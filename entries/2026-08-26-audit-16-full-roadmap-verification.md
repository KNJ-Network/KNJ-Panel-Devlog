# Phase 134 - Audit #16: Verifying Every "Live" Row Is Actually True

A different kind of audit than the last fifteen. Every prior one reviewed *new* code since the
previous pass — this one instead re-checked the entire public roadmap, all 178 rows marked "Live"
across both the Controller and account panels, against the real code, by code, not by memory. Seven
independent passes, each covering a slice of the roadmap, each required to cite the actual file and
line for every claim rather than trust the label already on the row.

It found four real problems worth fixing before the next release, plus seven stale or inaccurate
lines in the public roadmap copy itself — claims that were true when written and had quietly drifted
as the features around them changed.

## The real bug

**WebDAV's per-credential scoping didn't actually scope anything.** `WebdavAccountService` wrote
every credential for an account into one shared htpasswd file. nginx's `auth_basic_user_file` only
checks whether a submitted username:password pair exists *anywhere* in the file it's pointed at — it
never binds a specific credential to a specific `location` block. So any WebDAV login on an account
could authenticate against *any* of that account's WebDAV locations, including ones scoped to a
different subdirectory than the one that credential was supposed to be confined to. The per-credential
subdirectory scoping the feature advertised was cosmetic.

Fixed by giving each credential its own htpasswd file, keyed by the credential's own database id
(`webdav-{id}.htpasswd`), with each generated nginx `location` block referencing only its own
credential's file — the same one-file-per-credential shape `WebdavAccountService`'s own updated
docblock now explains. A stale shared file left over from before the fix is cleaned up the next time
that account's WebDAV config is touched. Two new tests prove the property directly: one confirms a
credential's own htpasswd file never contains another credential's hash, the other parses the
generated nginx config and confirms each `location /webdav/{id}` block references only its own file.

## What else turned up

- **Backup restore silently dropped FTP accounts.** The account backup/restore pipeline captures
  domains and databases into a manifest and restores them through the privileged provisioning
  script — but FTP credentials live in this app's own MySQL, not on disk, so they were never part of
  that manifest and a restore silently left them gone. Fixed with a parallel, PHP-owned side channel:
  `ftp-accounts.json` written directly into the backup directory alongside the script's own output,
  restored by upserting on username with the exact backed-up password hash (never regenerated —
  matches the byte-for-byte restore philosophy every other restorable thing in this codebase already
  follows). A new provisioning action, `ftp-account-restore`, writes the vsftpd side of it back without
  ever re-hashing the password.

- **The Visual Database Browser's table/row browsing ran as root**, unlike the query tool sitting one
  page over in the same feature, which already correctly authenticates as one of the account's own
  granted MySQL users. Table/row browsing used the same root-scoped provisioning actions the WHM admin
  side legitimately needs, but on the account side that meant any account could, in principle, browse
  the schema of *every* database on the server through its own UI. Fixed by mirroring the query tool's
  own pattern: two new `-as-user` provisioning actions, resolving the account's own granted database
  user the same way the query tool already does, with a clear "no database user has access yet" state
  instead of a silent root-scoped fallback when no grant exists.

- **PHP version selection only ever looked at the account's primary site.** An addon domain had no way
  to pin its own PHP version independently — `PhpVersionController::updateSite()` and the account panel
  above it both assumed one site per account. PHP *config* overrides (`.user.ini`) were already
  correctly scoped per-site through PHP's own per-directory mechanism; only the version *selector*
  hadn't caught up. Fixed by resolving the target site from a `site_id` the same way `SslController`
  already does (never trusting an implicit "always primary"), with a site-picker shown whenever an
  account has more than one domain. The UI copy for the PHP version toggle itself was also corrected —
  it's genuinely account-wide by architecture (one PHP-FPM pool per system user, shared by every domain
  on that account), not per-domain, and previously implied otherwise.

## The roadmap itself had drifted

Seven lines in the public roadmap no longer matched what the code actually does, none of them security
issues — just claims that were accurate the day they were written and hadn't been revisited since:

- **First-run setup** still described the old auto-generated-password install flow, from before the
  real setup wizard replaced it.
- **Server-wide settings** referenced a "Server Settings" page that no longer exists on its own — it
  was folded into Server Setup weeks ago.
- **Server role profiles** still called Mail Only "a real future server type, not yet built," after it
  shipped.
- **Repair mailbox permissions** described itself as domain-scoped; the actual privileged action takes
  no domain argument at all and repairs the whole mail store.
- **Directory login protection**'s description said "after repeated failed logins" while its own note
  right next to it correctly said the opposite — it locks on too many *distinct IPs succeeding* with
  the same login, not on failures.
- **Image tools** claimed a WebP conversion was "confirmed byte-for-byte with the file command" — `file`
  identifies a type by magic bytes, it doesn't do byte comparison, and a lossy format conversion was
  never going to be byte-identical to the source anyway. Softened to what was actually checked: the
  output really is a valid WebP file.
- **Calendars & contacts sync** claimed real-device auto-discovery confirmation that was never actually
  done — what was verified was the protocol itself (SRV records, `.well-known` redirects, a real
  PROPFIND/MKCALENDAR/PUT/GET round trip), not an actual iOS or Thunderbird client connecting.

All seven corrected in the public roadmap; the "Choose PHP version for a domain" line was corrected
alongside them for the same account-vs-domain scope issue as the code fix above.

## Verification

Every fix has real, non-mocked test coverage where the fix itself is testable without root — a real
tar-based round trip for backup browsing (already covered before this audit, confirmed still accurate
rather than assumed), a real cross-credential htpasswd assertion for the WebDAV fix, a real DB-write
assertion for the FTP restore fix, and process-argv assertions proving the database browser genuinely
authenticates as the scoped user rather than root. The privileged, sudo-gated halves of backup/restore
stay `Process::fake()`-verified, same as every other privileged action in this codebase — there's no
way to exercise real root-owned multi-user filesystem operations from a test suite, and this codebase
has never pretended otherwise.

Also closed two coverage gaps the audit flagged even though they weren't bugs: the Apply Update path
for the panel's own self-upgrade had zero test coverage until now (service, job, and controller all
covered, including the "already running" guard), and raw access log downloads on the account side had
no tests at all despite the WHM-side equivalent being fully covered. ionCube's one-click install was
also live-triggered for real against panel.dev.knj.network's PHP 8.5 and confirmed loaded via the
page's own live `php -m` check, not just a database flag.

Full suite (2,009 tests) and `pint --test` both clean.

## Re-audit: round two

Per standing instruction, the same seven-way independent verification was re-run from scratch against
the deployed fixes above — same method, no memory of the first pass's findings carried in, everything
re-derived from the code as it now stood. It came back with one more real bug and eleven more stale
roadmap lines.

**The PHP version fix above was incomplete.** `PhpVersionService::setSiteVersion()` correctly resolved
*which* site to target via `site_id` instead of always assuming primary, but it then only updated that
one site's own database row. The underlying constraint is the same one already documented above — one
PHP-FPM pool per system user, shared by every domain on the account — so switching site A to PHP 8.2
and then site B to PHP 8.3 left site A's database row still claiming 8.2 while the one real, shared
pool (and thus what site A actually serves) was 8.3. Fixed by having the update apply to every site on
the account, not just the one passed in, so the stored state can never drift from the one real version
being served. The WHM copy was tightened further to say so explicitly. A regression test creates two
sites on one account, pins one, and asserts both rows come out in sync.

**Eleven more roadmap lines had drifted**, same shape as the first seven: claims that were true when
written and quietly stopped being true as the surrounding features changed. Mail routing, certificate
upload, database/user management, and per-account PHP configuration notes all implied a Controller-side
counterpart to the account-side feature that was never actually built — corrected to say so plainly.
Per-package feature control's toggleable-feature count was stale (26, actually 34). Three separate
notes still referenced a "Server Settings" page from before it was folded into Server Setup — the first
pass's fix for Timezone configuration's Server Setup reference apparently missed its two siblings (DNS
resolver configuration, Remote reboot), now all three are consistent. Custom error pages and custom
MIME types were still described as "per domain" — like the earlier PHP scope issue, both are genuinely
account-wide (applied to the primary domain's vhost), not independently configurable per addon domain.

**One coverage gap remained**: `SystemRebootController::store()` had no test at all — the existing
`ServerSetupTest` only asserted the button's label text rendered on the page, never submitted it or
checked that it actually calls the privileged script. Added both the success path (asserts
`reboot-server` reaches the provisioning script) and the failure path (asserts the error surfaces in
the session flash instead of silently claiming success), plus the same non-admin/reseller-forbidden
checks every other WHM action in this codebase carries.

Full suite (2,014 tests, up from 2,009) and `pint --test` both clean. Live-verified: reloaded
panel.dev.knj.network's PHP Manager page post-deploy and confirmed the corrected "one shared process
per account" copy is live.

A further re-audit round against this fix set is in progress before any version bump or release cut.

## Re-audit: round three

The same seven-way independent check ran again, fresh, against round two's deployed fixes. It found
two more real bugs, four more stale roadmap lines, and one more coverage gap — nothing left unfound
from the first two rounds, but each round's own fixes opened a small new surface the next round
caught.

**The account-wide PHP version fix was still incomplete — new sites didn't inherit it.** Round two
fixed `setSiteVersion()` to keep every *existing* site on an account in sync with the one real shared
FPM pool. What it didn't cover: a *new* addon domain or subdomain added after an account had already
switched away from the default version. `DomainsController::store()` and `SubdomainService::create()`
both built the new `Site` row with `php_version_id` left null — "inherit/default" — even on an account
already pinned to something else. No functional outage (the shared pool still serves the new site
correctly either way), but the new site's own database row, and both the WHM and account-side PHP
selectors reading it, would misreport the version actually in use. Fixed by having both creation paths
inherit whatever the account's existing sites are already pinned to, rather than defaulting to null.

**The MariaDB upgrade snapshot was never actually mandatory.** The upgrade page has always described
its pre-upgrade snapshot as "mandatory" — in the roadmap copy and in the confirm dialog — but nothing
in `DatabaseUpgradeController::apply()` enforced it. An admin could trigger a major-version upgrade
with zero completed snapshots on record, meaning no rollback path in place despite what the UI
promised. Fixed by having `apply()` actually require a completed snapshot to exist before dispatching
the upgrade job.

**Four more roadmap lines had drifted.** "Server-wide settings" still pointed contact/notification
emails at the Server Setup page after they moved to the dedicated Notifications page. The client-side
"Per-domain mail routing" note claimed an admin could configure routing for any domain from the
Controller side — no such action, controller, or view exists anywhere in the codebase; only the
admin-side row (fixed in round two) needed that disclosure, this client-side row had the same false
claim independently. "Directory listing style" implied a choice between multiple listing styles when
the feature is a plain per-folder on/off toggle. "WebDAV access" claimed every account gets a
full-access WebDAV login created automatically — it's 100% self-service; nothing is auto-provisioned,
the owner creates each login themselves.

**One more coverage gap**: Directory password protection has real cross-account ownership checks
(`ensureOwnsProtection()`) and a real nginx-injection charset guard on the protected path (from audit
#11), but neither had a test proving it. Added cross-account tests for unprotect/add-user/remove-user,
and a regression test for the injection guard using the exact kind of folder name (`evil'} location
/x {alias /etc;} location /{y`) the guard exists to reject.

Full suite (2,022 tests, up from 2,014) and `pint --test` both clean. Live-verified: reloaded the
Database Upgrade page on panel.dev.knj.network post-deploy, loads clean with no errors.

A fourth re-audit round against this fix set is in progress before any version bump or release cut.

## Re-audit: round four

The same seven-way check ran a third time against round three's deployed fixes. It came back with
three findings — two real bugs and one deliberate design choice made visible rather than fixed.

**Blocked sender domains raced Mail Settings, same shape as the recipient bug fixed in the very first
pass.** `mail-filter-domains-write` set `smtpd_sender_restrictions` directly from the provisioning
script, instead of only writing the map file and letting `MailServerSettingsService::apply()` own the
directive — meaning a domain-block save could silently overwrite whatever the last Mail
Settings/Greylisting/Spam Filter save had composed, dropping `permit_mynetworks`/
`permit_sasl_authenticated` or an enabled `reject_unknown_sender_domain`. Fixed by mirroring the
recipient path exactly: the script only writes and postmaps the map file now, and
`MailFilterService::updateBlockedDomains()` calls `apply()` immediately after, which now also composes
`check_sender_access` into `smtpd_sender_restrictions` when domains are blocked.

**Convert Addon Domain to Account carried over a stale PHP version.** The provisioning script's new
FPM pool for the converted account is always created at the hardcoded system default version
(`FPM_POOL_DIR="/etc/php/8.5/fpm/pool.d"`) — never at whatever the addon site happened to be pinned to
on the old account. A site previously switched to a non-default PHP version would convert into a new
account whose database row kept claiming that version, while the real pool serving it was always the
default. Fixed by resetting the new primary site's `php_version_id` to null on conversion, matching
what's actually provisioned.

**The mandatory snapshot gate (round three) checks existence, not freshness — made visible, not
force-fixed.** A snapshot from months ago satisfies the same "does one exist" check as one from five
minutes ago. Rather than invent an arbitrary age cutoff and block on it, added a non-blocking amber
warning on the Database Upgrade page when the most recent snapshot is 24+ hours old, so staleness is
visible to the admin deciding whether to snapshot again before a real upgrade — the actual judgment
call stays with them.

Full suite (2,026 tests, up from 2,022) and `pint --test` both clean. Live-verified: reloaded the
Database Upgrade page on panel.dev.knj.network post-deploy and confirmed the new staleness warning
renders correctly against the real week-old snapshot already on that box.

A fifth re-audit round against this fix set is in progress before any version bump or release cut.

## Re-audit: round five

Six of the seven batches came back completely clean this time — every item CONFIRMED, including
independent re-verification of all three round-four fixes (the sender-domain race, the Convert Addon
Domain PHP reset, and the snapshot staleness warning). The seventh batch found one thing: Calendars &
contacts sync's roadmap note has claimed since it shipped that ".well-known resolution" was "verified"
alongside the DNS SRV records and the protocol-level PROPFIND/MKCALENDAR/PUT/GET round trip — but no
test anywhere actually proved `NginxSettingsService` writes the `/.well-known/caldav` and
`/.well-known/carddav` redirects. The sibling `/webmail`, `/account`, and `/controller` shortcuts,
written by the same method right next to these two lines, have had exactly this kind of test since
they shipped; the CalDAV ones never got the same treatment. Not a functional bug — the redirects
themselves were already correct, live-verified with real `curl` requests when CalDAV/CardDAV first
shipped — just a real, specific gap between what the roadmap claimed was tested and what the test
suite actually covered. Added both the presence case (redirects appear once `panel.hostname` is known)
and the omission case (nothing written before then), mirroring the existing webmail/account/controller
tests exactly.

Full suite (2,028 tests, up from 2,026) and `pint --test` both clean.

A sixth re-audit round against this fix set is in progress before any version bump or release cut.

## Re-audit: round six

Five findings this round — the two real bugs of a new class this audit hadn't caught before, two
private-doc scope overclaims, and one more test-coverage gap.

**Two admin settings pages could silently drift from what the server actually had applied.**
`EmailArchivingService::update()` and `DatabaseConfigService::update()` both persisted the admin's
chosen values to `Setting` *before* confirming the privileged provisioning-script call had actually
succeeded. A failed write — Postfix's bcc-maps config for archiving, my.cnf for database tuning —
still left the WHM page reporting the new values as current, even though nothing on the live server
had changed. This is the exact bug class audit #15 already fixed once for `ClamavService`, and this
session's earlier rounds fixed again for `TlsPolicyService` — both now follow that same
persist-only-after-success order, with regression tests proving Settings stay unchanged when the
script call fails.

**Two private roadmap.md claims overstated scope the code never had.** IP Blocker and Optimize
Website were both documented as covering "any of the account's sites" / "per-domain," but both
services only ever write their nginx directive into the primary domain's vhost — the same
account-wide-only scope already established (and already disclosed) for Directory Listing, Custom
Error Pages, and Custom MIME Types elsewhere in this same document. Corrected the copy to match, and
fixed the matching public roadmap.json overclaim for Website compression's description.

**The Nameserver Report's reseller scoping had real code but no test proving it.** The scoping logic
in `DnsController::nameserverReport()` was already correct and had been for a while, but the only
reseller-related test on this page checked that a plain account user gets 403 — never that a reseller
genuinely only sees their own domains, unlike the sibling Deliverability feature's own test of this
exact scoping. Added the matching test.

Full suite (2,030 tests, up from 2,028) and `pint --test` both clean. Live-verified: reloaded the
Email Archiving page on panel.dev.knj.network post-deploy, loads clean with no errors.

## Re-audit: round seven

The seventh round of the 7-agent audit turned up the biggest single finding of this whole
verification pass: the "persist Settings/a DB row before confirming the privileged provisioning
script actually succeeded" bug — the exact class fixed for `ClamavService` and `TlsPolicyService` in
earlier audits, and for `EmailArchivingService`/`DatabaseConfigService` in round six above — was
never actually fixed everywhere. It was present, unfixed, in roughly thirty more services spanning
almost the entire mail-configuration subsystem plus most of the account-side per-domain site toggles:
`DnsClusterController`, `MailSettingsController`, `SpamFilterService`, `SpamdSettingsService`,
`MailRelayController`/`MailRelayService`, `MailServerConfigController`, `MailFilterService`,
`GreylistingService`, `GreylistExemptionService`, `FtpSettingsController`, `LogRotationController`,
`WebdavAccountService`, `FirewallService`, `WafService`, `SystemCronJobService`, `CronJobService`,
`AutoresponderService`, `MailRuleService`, `MailboxService::createForwarder()`, `SpamScoringController`,
`ChallengeResponseService`, `DnsController` (account-side Zone editor), `DynamicDnsService`,
`IpBlockerService`, `HotlinkProtectionService`, `LeechProtectionService`, `OptimizeWebsiteService`,
`DirectoryIndexService`, `ErrorPageService`, and `MimeTypeService`. In every case, a failed script
call left the panel UI reporting a change as applied when the real server-side config, vhost snippet,
crontab entry, or sieve script was never actually touched.

A handful of these were worse than a stale-looking UI. `LeechProtectionController` only caught
`FileManagerException`, not the `RuntimeException` a failed sync actually throws, so a real failure
was an uncaught 500 with the DB already changed underneath it. `CronJobController::destroy()`
(account-side) had no try/catch at all around a call that can throw — a failed cron-sync deleted the
job from the database while the real crontab entry survived, orphaned, with no UI left to find or
remove it. `DynamicDnsService::deleteHost()` had the same shape: a failed zone write could leave a
live A record on the wire with no DB row and no UI control pointing at it anymore. `WafController`'s
settings-save action had no try/catch either, so a failed apply surfaced as a raw 500 instead of an
error flash.

Two different fix shapes were needed depending on how each script call actually reads its content.
Where a service builds the script's input purely from the method's own parameters (the
`ClamavService`/`TlsPolicyService` shape), the fix was a straight reorder — run the script, confirm
success, only then persist. Several services turned out to have their `apply()`-style method
re-reading `Setting::get()` for the very values being saved, which would have made a naive reorder
persist-then-apply-stale-values instead of fixing anything — `DnsClusterService`, `MailServerSettingsService`,
`MailRelayService`, and `MailServerConfigService` were refactored so `apply()` takes the new values as
parameters instead. Where the privileged script itself queries the DB row being written (most of the
per-domain nginx-snippet services, and the DNS/DynamicDNS zone writers), a clean pre-persist reorder
isn't possible — those were wrapped in `DB::transaction()` so a failed script call rolls the write
back, following the precedent `SystemCronJobService`/`MailRuleService` already established for exactly
this case. `WebdavAccountService` and `FirewallService` needed explicit rollback-and-cleanup (deleting
the just-created row and any file it wrote) rather than a transaction, since the row has to exist on
disk before the script that validates it can run.

Every one of the ~30 fixes shipped with a regression test that fakes a failing script call and asserts
the Setting/DB row stayed at its prior value — including renaming two existing tests
(`FtpSettingsTest`, `LogRotationTest`) that had literally asserted the buggy behavior as correct
(`test_a_failed_apply_shows_an_error_but_still_saves`) into ones that assert the fixed behavior
instead. Alongside the main fix wave: `InitialSetupController::storeBasics()` was checking nothing
about whether its `set-hostname`/`set-timezone` script calls actually succeeded, silently letting the
setup wizard advance and permanently lock `setup.completed_at` even on failure — now checked and
blocking; `SslControllerTest` had zero coverage of the custom-certificate upload flow at all (the
underlying code was already correct) — added; and four more `docs/roadmap.md` bullets (Hotlink
Protection, Directory Listing Style, Custom Error Pages, Custom MIME Types) plus the public
roadmap.json's WebDAV access note needed the same primary-domain-only scope disclosure already applied
to their siblings in earlier rounds.

Five parallel fix agents split the ~30 files with no overlap, each adding its own regression tests and
confirming its own slice of tests passed before handing back. Full suite (2,081 tests, up from 2,030)
and `pint --test` both clean after merging all five. Live-verified: reloaded the Mail Relay and WAF
settings pages on panel.dev.knj.network post-deploy, both load clean with no errors.

## Re-audit: round eight

The eighth round of the 7-agent audit re-verified all of round seven's ~30 fixes (every one confirmed
genuinely correct, with real fake-failure tests) but also found that the same persist-before-script-
success bug pattern was still present, unfixed, in a family of mail services that call
`AuthMapService::regenerate()`/`ensureInfra()` rather than a direct `scriptRunner->run()` call — a
slightly different shape that round seven's targeted greps missed entirely. This mattered specifically
for installs with a linked Mail Only satellite server: a failed push there could leave the panel's own
database out of sync with what the satellite mail server was actually enforcing.

Affected: 7 of `MailboxService`'s 8 mutating methods (`createMailbox`, `deleteMailbox`,
`changePassword`, `updateQuota`, `suspend`, `unsuspend`, `deleteForwarder` — only `createForwarder`
had already been fixed, in round seven), `DefaultAddressService::setCatchAll()`/`clearCatchAll()`,
all four of `MailingListService`'s mutators, and `MailFilterService`'s blocklist methods (whose second,
enforcement-rebuilding script call could fail after the first had already succeeded). All wrapped in
`DB::transaction()`, since in every case the `regenerate()`/`apply()` call reads the row being written
and can't simply run first. Two related gaps closed alongside: `MailController::suspendMailbox()`/
`unsuspendMailbox()` and its bulk-forwarder-import loop had no try/catch at all around calls that can
now throw — a satellite-push failure was an uncaught 500 (or aborted the whole CSV batch) rather than a
flashed error or a skip-and-continue; same gap found and fixed in
`DefaultAddressController::destroy()` while wrapping up this fix wave.

Four more instances of the same underlying anti-pattern turned up outside the mail-auth-map family,
in services round seven's sweep never reached:

- **`DirectoryPrivacyService::protect()` — the most serious of the four.** It created the DB row and
  wrote the htpasswd file *before* the privileged nginx-config sync confirmed success. A failed sync
  left a phantom "protected" row: the account owner's UI said the directory was password-protected
  when nginx had never actually been told to enforce it — a real fail-open security gap, not just
  stale-looking UI. Fixed with the same rollback-on-failure shape `WebdavAccountService::create()`
  already used (delete the row, unlink the htpasswd entry, on sync failure).
- `DatabaseManagerService::allowRemoteHost()` — same shape but fail-closed (a failed firewall sync
  just means the port never actually opens), fixed the same way.
- `CronJobService::updateCronEmail()` had been missed when `create()`/`delete()` got the
  `DB::transaction()` treatment in round seven; `CronJobController::updateEmail()` had no try/catch
  either, so a failed sync was both an uncaught 500 and a silently-wrong `cron_email` in the database.
- `QuickSiteService::publish()` persisted the `QuickSite` row without ever checking whether
  `file_put_contents()` actually succeeded — fixed by checking the write first and throwing before any
  DB write on failure.

Two smaller, related UX gaps in `AccountController` were fixed alongside rather than left for a future
round: a failed disk-quota re-application on package change was only `report()`'d (logged) and never
shown to the admin, who'd see "Account updated." even though enforcement hadn't actually changed —
now flashes an error too, without blocking the metadata update itself; and `suspend()`/`unsuspend()`
had no try/catch around their provisioning calls, so a script failure was a raw 500 instead of a
graceful redirect.

One subtle bug surfaced while building the `MailFilterService` fix: `Setting::get()`'s
`Cache::rememberForever()` isn't transaction-aware, so wrapping a `Setting::set()` plus a call that
reads that same setting inside one `DB::transaction()` meant a rollback reverted the database row but
left the cache still holding the new, never-committed value. Fixed with an explicit `Cache::forget()`
in the catch block for the two methods this affected.

Two fix agents split the ~10 remaining files with no overlap. Full suite (2,108 tests, up from 2,081)
and `pint --test` both clean. Live-verified: reloaded the Default Address and Directory Privacy pages
on panel.dev.knj.network post-deploy, both load clean with no errors.

## Re-audit: round nine

The ninth round found the same "persist before confirming a privileged action succeeded" bug had two
more forms nobody had caught yet, plus a batch of smaller uncaught-exception gaps discovered by
cross-referencing each fixed controller against its unfixed sibling.

**`GreylistingService` had the identical bug already fixed for `MailFilterService`, but was itself
missed.** Both persist a Setting, then make a second privileged script call that rebuilds Postfix's
`smtpd_recipient_restrictions` directive — if that second call fails, the Setting stays "new" with
nothing to roll it back. `MailFilterService` got this fixed correctly in round eight (`DB::transaction`
+ an explicit `Cache::forget()`, since `Setting::get()`'s forever-cache isn't transaction-aware);
`GreylistingService` has the exact same shape and was never touched. Fixed identically.

**The entire admin DNS Zone Manager subsystem had never received this treatment at all** — a whole
feature area every prior round missed, because its privileged action (`writeZone()`, BIND validation
+ write) is structurally different from the `scriptRunner->run()`/`AuthMapService::regenerate()`
shapes every earlier fix targeted. `DnsController`'s `store()`/`update()`/`destroy()`/`bulkUpdate()`,
`DnsZoneService`'s `restoreDefaultRecords()` and `importRecords()`, and `DnsClusterService`'s
`generateKeyFor()` all wrote a DnsRecord (or, for the cluster case, a TSIG key onto the Server row)
before confirming the write actually landed in BIND, with no rollback — and this drift is invisible to
the existing "Missing Zones" repair tool, which by its own doc comment only detects a zone being
entirely absent, not content drift within one that still exists. All five wrapped in `DB::transaction()`,
mirroring the account-side `DnsController`'s own already-correct shape.

The worst instance was `backfillMailAuth()` (used by "Bulk-enable mail signing"): it didn't even catch
the exception at all, so one zone failing mid-batch was an uncaught 500 with every earlier zone in the
batch already committed. It now wraps each zone's backfill in its own transaction and try/catch and
reports "N updated, M failed" instead of either silently succeeding or crashing the whole request.

Four more gaps were found the same way this whole audit keeps finding them — reading a fixed
controller next to its unfixed twin: `HotlinkProtectionController::destroy()` was the one sibling of
Directory Listing/Custom Error Pages/MIME Types/Leech/IP Blocker's `destroy()` actions that never got
the `catch (FileManagerException|RuntimeException)` treatment; the admin-side
`SystemCronJobController::destroy()`/`updateEmail()` never got the fix its account-side sibling
`CronJobController` already had; and `SecurityScanController::run()` and `SslController::generateCsr()`
both call something that can throw with no catch at all, each now redirecting with a flashed error
instead of a raw 500.

Two fix agents split the files with no overlap — 21 files, 5 real bugs plus 4 missing-catch gaps, 15
new regression tests. Full suite (2,123 tests, up from 2,108) and `pint --test` both clean. Live-verified:
reloaded the Greylisting and Zone Templates pages on panel.dev.knj.network post-deploy, both load clean
with no errors.

## Re-audit: round ten

By round ten, the audit had converged on a specific, repeatable technique for finding the last of these
bugs: for every feature, compare its already-fixed "create"/"store" action against its "delete"/
"destroy" sibling — repeatedly, the create side had been fixed in an earlier round while the delete side,
same file, same shape, was simply never checked. Every finding this round came from that one comparison,
applied systematically across the whole codebase.

**Missing try/catch** (an uncaught `RuntimeException` on a privileged-script failure, where the sibling
action already had one): `SslController::generateCsr()` on the account side — the customer-facing twin
of the WHM controller fixed in round nine, missed entirely; `MailController::destroyForwarder()`,
missing what `destroyMailbox()` in the same file already had; `MailingListController::destroy()`/
`destroyMember()`, missing what `store()`/`storeMember()` already had; `DynamicDnsController::destroy()`;
`QuickSiteController::store()`/`destroy()`; `ProcessMonitorController::index()`, missing the graceful-
degradation pattern `MailQueueController`/`MailDeliveryController`/`MailStatisticsController` already
used for the identical "render live external-command data" shape; and `BackupHistoryController::browse()`
/`BackupController::browse()`, missing what their own sibling `restore`/`export` actions in the very same
files already had.

**Wrong-order persist-then-verify** (the delete/revoke path doing the opposite of its own already-fixed
create/approve path — deleting first, confirming the privileged write second): `ChallengeResponseService::
revoke()` (opposite of `approve()`/`setEnabled()`), `QuickSiteService::unpublish()` (opposite of
`publish()`), `DirectoryPrivacyService::unprotect()` (opposite of `protect()`), `WebdavAccountService::
delete()` (opposite of `create()`), and account-side `DnsController::bulkUpdate()` — missing the
`DB::transaction()` wrap its own `store()`/`update()`/`destroy()` in the same file already had.

Two more turned up in `ServerController`, both in the DNS-clustering TSIG-key flow: `completeLink()`
persisted the newly-linked hostname/IP/status before confirming `regenerateClusterConfig()` succeeded —
fixed the same way `DnsClusterService::generateKeyFor()` was in round nine, wrapping both in one
transaction. `destroy()` had the identical ordering issue, but removing a server is a genuine,
non-reversible admin action — rolling back the deletion because a config-regenerate afterward failed
would be wrong. Instead it now lets the deletion stand and flashes a warning if the regenerate fails,
the same "let the primary action succeed, surface a warning for the best-effort secondary one" shape
`AccountController::update()`'s quota-reapply fix used in round nine.

Two fix agents split the 13 findings across ~33 files with no overlap. Full suite (2,141 tests, up from
2,123) and `pint --test` both clean. Live-verified: reloaded the Resource Monitor and Servers pages on
panel.dev.knj.network post-deploy, both load clean with no errors.

## Re-audit: round eleven

Round eleven found fewer bugs than any round since the create-vs-delete-sibling technique was adopted —
five instances, down from thirteen in round ten — which is the trend that matters more than any single
round's count: 30 → 9 → 13 → 13 → 5. The finds themselves stayed real, though, including the single
worst bug this whole audit has turned up.

**`FirewallService::remove()` was fail-open.** `add()` (already correctly fixed) creates the
`FirewallRule` row, syncs the ufw config, and rolls the row back if the sync fails. `remove()` did the
exact opposite, in the wrong order: it deleted the row *before* calling `sync()`, with no rollback at
all. If that sync failed — the ufw rule set never actually rewritten — the database already showed the
IP as removed while the real firewall kept allowing it through. Every other instance of this bug class
found across eleven rounds left the panel *overclaiming* that a protection was in place; this one
undersold what was actually still blocked, which is the more dangerous direction. Fixed with the same
snapshot-delete-sync-restore-on-failure shape already used by `WafService::setSiteMode()`, with a
regression test that fakes a failed sync and asserts the original rule is still present in the database
afterward.

Four smaller instances of the same family, all found by the same by-now-standard comparison of a fixed
create/enable action against its never-checked delete/disable sibling: `SubdomainService::create()`/
`destroy()` were missing the `DB::transaction()` wrap its direct sibling `DynamicDnsService` already had;
`ApiTokenController::destroy()` was missing the `ensureFeatureEnabled('api_tokens')` check its own
`index()`/`store()` already had, letting a customer whose package no longer includes API tokens still
revoke them; `DatabaseManagerService::revokeRemoteHost()` had the identical asymmetric-rollback shape as
the firewall bug (lower severity here, since the underlying MySQL credential revocation already happens
first — a failed sync only leaves a stale, credential-less firewall allow-rule, not live unauthorized
access); and `TransferToolController::destroySource()` turned out to already be correct, just entirely
untested.

One fix agent (this round's finding set was small enough not to need two) covered all five, full suite
(2,148 tests, up from 2,141) and `pint --test` both clean. Live-verified: reloaded the Access Control
page on panel.dev.knj.network post-deploy, loads clean with no errors.

## Re-audit: round twelve

Round twelve ran all seven batches with fresh-eyes instructions rather than mere re-confirmation of prior
verdicts, and turned up two more real findings — both from the same "compare a fixed create/delete
sibling against every other unchecked mutating pair" technique, applied to two call sites the prior eleven
rounds hadn't reached yet.

**`Api\DnsRecordController::store()`/`destroy()`** had the identical persist-before-verify bug as the
account-side web `DnsController` and `SubdomainService::create()`/`destroy()` — both already fixed in
earlier rounds — except this one lived in the token-authenticated public API surface, which none of the
prior rounds' file lists happened to cover. `store()` created the `DnsRecord` row, then called
`writeZone()`; `destroy()` deleted it, then called `writeZone()`. Either way, a failed BIND write (bad
zone data, `named-checkzone` rejecting it) left the database permanently out of sync with the live zone,
with only a 422 response and no rollback. Fixed by wrapping both in `DB::transaction()`, mirroring
`DnsController::store()`/`destroy()` exactly. Two regression tests fake a failing `dns-zone-write` and
assert the create rolls back / the delete never happens.

**`AddonDomainConversionController::create()`/`store()`** never checked the `accounts.create` reseller
permission that every other account-creation path (`AccountController::create()`/`store()`) already
enforces — a reseller with that permission revoked could still create a brand-new account by converting
one of their own addon domains, since "Convert to Account" creates a real new `Account` row under the
hood. The "Convert to Account" link on the Parked Domains list was also unconditionally shown regardless
of the permission. Fixed by adding the same `abort_unless(hasResellerPermission('accounts.create'))`
check both actions' siblings already have, and hiding the nav link behind the same check. Two regression
tests cover both the form and the submit path being blocked for a reseller with the permission revoked.

Full suite now at 2,152 tests (up from 2,148), `pint --test` clean. Live-verified: reloaded the Parked
Domains page on panel.dev.knj.network post-deploy, loads clean with no errors.

## Re-audit: round thirteen

Round thirteen's roadmap-row list also caught up on a genuine gap in the audit's own bookkeeping: "KNJ
Webmail (custom client)" — the flagship from-scratch webmail client this project is largely built around
— had somehow been missing from every batch's item list since round nine, a row-extraction bug in the
audit's own tooling rather than anything wrong with the feature. It got a first real audit pass this
round: CONFIRMED, with real cross-account authorization checks (messages/attachments are never
DB-owned — resolved live via per-session IMAP credentials, so cross-account access is structurally
prevented by the mail server's own auth; contacts and filing rules, which are DB-backed, have explicit
ownership checks with dedicated tests) and roughly 900 lines of feature test coverage across compose,
drafts, bulk actions, folders, search, sort, CSV import, and HTML sanitization.

Two real findings, both in the DNS/mail-provisioning layer:

**`DnsZoneService::createZoneForSite()`** — the primary zone-creation path, called on every new account
and every new domain — created the `DnsZone` row and every one of its records before `writeZone()`
confirmed the BIND write actually succeeded, with no rollback. The exact same file already had three
sibling methods (`restoreDefaultRecords()`, `ensureMailAuthRecords()`, `importRecords()`) correctly
wrapping this scenario in `DB::transaction()`, each with a comment explaining why — the zone-creation
path itself was simply never brought in line. A failed write left a zone the database reported as fully
configured that was never actually pushed to BIND, silently, since both callers (`ProvisionAccountJob`
and the account-side `DomainsController`) already treat this as best-effort and just log the failure.
Fixed the same way as its siblings; a new regression test fakes a failing `dns-zone-write` and asserts
neither the zone row nor any of its records survive.

**`dovecot-auth-sql.conf.ext.2.4.template`** — the more serious of the two, and the kind of bug this
audit exists to catch: a deploy-script gap rather than an application-code one, invisible to the PHP
test suite entirely. The 2.3 template (`dovecot-sql.conf.ext.2.3.template`) filters suspended mailboxes
out of its passdb login query and returns a `quota_rule` field from its userdb query; the Mail Only
satellite push (`MailAuthMapService`) does the same, in its own passwd-file format. The 2.4 rewrite —
used automatically on any Main server running Dovecot 2.4, which `setup-mail-server.sh` targets on
newer Ubuntu releases — dropped both. Net effect: on a 2.4 Main install, suspending a mailbox from the
panel updated the database but the mailbox could still log in and receive mail over IMAP/POP3 exactly
as before, and no per-mailbox quota was ever enforced regardless of what was configured. Fixed by
adding the identical `AND m.suspended = 0` filter and `quota_rule` `CASE` expression to the 2.4 query,
plus a comment pointing at the 2.3/satellite precedent so a future rewrite of this file doesn't drop
them a third time. No Dovecot-2.4 server exists yet to live-verify the mail-serving behavior itself
against (`panel-dev` runs 2.3) — verified by direct comparison of the generated SQL against the
already-proven 2.3/satellite queries instead; live verification against a real 2.4 install is still
owed once one exists.

Full suite now at 2,153 tests (up from 2,152), `pint --test` clean. Live-verified: reloaded DNS Zones
on panel.dev.knj.network post-deploy, loads clean with no errors.

A fourteenth re-audit round against this fix set is next before any version bump or release cut.

## Re-audit: round fourteen — clean

Round fourteen came back with zero findings across all seven batches. Every item independently
re-derived from current code, not from any prior round's verdict — including a full fresh-eyes second
pass on "KNJ Webmail (custom client)" (nothing new since its first real audit last round) and explicit
re-verification, with the actual test suites re-run rather than trusted from a summary, of both of round
thirteen's fixes: `DnsZoneService::createZoneForSite()`'s transaction wrap (confirmed `dnsOnlySync
->pushToAll()` is structurally unreachable on a rollback, since it sits after the `DB::transaction()`
call returns and a rollback rethrows) and the Dovecot 2.4 template's suspended-filter/quota_rule fields
(confirmed field-for-field identical to the 2.3 template and the Mail Only satellite push).

This is the first fully clean pass since this audit began — thirteen re-audit rounds after the initial
sweep, each one narrower than the last:

```
round:    initial  6   7   8   9  10  11  12  13  14
findings:   many   4  ~30  9  13  13   5   2   2   0
```

## What this audit was, end to end

Started as a check on whether every roadmap row marked "Live" on the public site was actually, truly
shipped — not from memory, from the real code, file and line, batch by batch. Found the single worst bug
of the whole run almost immediately (WebDAV's shared-htpasswd-file scoping gap, a real fail-open), then
kept finding smaller instances of the same underlying pattern — a privileged provisioning-script or DNS
write committed to the database *before* confirming it actually succeeded, with no rollback — spread
across dozens of otherwise-unrelated features, because the codebase had grown that convention gradually
rather than having it enforced everywhere from day one. The technique that found the long tail of
these — compare every "create"/"enable" action's already-fixed shape against its "delete"/"disable"
sibling, which had usually been overlooked — only crystallized partway through (round nine or so) and,
applied systematically afterward, is what actually closed this out rather than an ever-growing list.

Two genuinely severe findings stand out from the rest: `FirewallService::remove()` (round eleven) was
the one instance across the whole audit where the bug direction was reversed — every other case *oversold*
protection that had silently failed; this one *undersold* it, deleting a firewall rule from the database
before confirming the real `ufw` sync succeeded, so a failed sync left the panel showing an IP as blocked
while the live firewall still let it through. And the Dovecot 2.4 template gap (round thirteen) — a
deploy-script bug invisible to the PHP test suite entirely, where a suspended mailbox could still
authenticate and no quota was ever enforced on any Main install running the newer Dovecot generation.

Full suite: 2,153 tests, all green. `pint --test`: clean. `panel-dev` confirmed clean and ready to serve
as the source for a new production release. **v0.16.74 cut and published to `KNJ-Panel-Builds` from this
confirmed-clean state.**

## Re-audit: round fifteen — a different bug class, and why that matters

One more confirmation pass was run after the v0.16.74 cut, on the same seven-way methodology as every
round above. It found two more real bugs — both structurally different from every bug this audit had
caught in the fourteen rounds before it.

**Perl Module and PEAR Package installers left their run stuck in `Running` forever on a timeout.**
`Process::timeout(120)->run(...)` in both `PerlModuleService::install()` and
`PhpPearPackageService::install()` only had failure handling for an *unsuccessful result* — but a real
timeout doesn't return an unsuccessful result, it throws `Illuminate\Process\Exceptions\
ProcessTimedOutException`, skipping the failure-recording block entirely. The stuck run permanently
blocked every future install of that type too, since the "already running" gate both controllers check
is server-wide, not per-account. Fixed by wrapping both calls in `try/catch (Throwable)`, mirroring the
shape `AppInstallerService::runInstall()` already used correctly.

**New account provisioning never synced the new site's PHP version against the current default.**
`PhpVersionService::setDefault()` only flips a DB flag — it has no effect on any live server state, by
design, since the provisioning script's `FPM_POOL_DIR` is what actually determines which PHP a new pool
gets built at, and that path is hardcoded to `8.5`. Currently latent (the seeded default happens to match
the hardcoded value), but a real bug the moment an admin changes the default — every new account from
then on would provision onto the old version while its database row and the admin's own dashboard
reported the new one. Fixed by having `ProvisionAccountJob` best-effort sync the new site to the current
default right after activation, reusing the same `PhpVersionService::setSiteVersion()` mechanism a manual
PHP Selector switch already uses.

Full suite now at 2,153 tests (unchanged — one new test file's assertions extended rather than a new
file added), `pint --test` clean. Live-verified: deployed commit confirmed live on `panel.dev.knj.network`
via direct SSH check of `/srv/panel`'s HEAD against the pushed commit, queue worker confirmed restarted
cleanly by auto-deploy and running the new job signature.

**Why this round matters more than its bug count suggests.** Both bugs belong to a class none of the
prior fourteen rounds' technique could have found: an *uncaught exception bypassing failure-handling
code entirely*, as opposed to code that runs to completion but writes state in the wrong order (the
create/delete-sibling, persist-before-verify, missing-try-catch shapes that closed out rounds six through
fourteen). A codebase can be completely clean by one detection technique and still hide bugs from a
structurally different one. Two agents independently reviewing the same file (Perl Module installer) gave
contradictory verdicts here — one saw a try/catch existed and called it fixed without checking what it
actually caught; the other looked closer and found it was the wrong exception type entirely — a reminder
that even within one technique, verification depth varies and disagreement between agents is a signal to
dig deeper, not average out.

This is the reason the next audit round is a genuine methodology change rather than another repeat of
rounds six through fourteen's roadmap-row sweep: a bug-class-organized pass across the whole codebase —
uncaught-exception gaps, transaction completeness, authorization/IDOR, injection surfaces, stuck-job/
non-terminal-state bugs — each scanning every file that shape could apply to, not a slice of roadmap rows.
The codebase isn't considered launch-ready until that comes back clean too.

## Round sixteen — the methodology change, run for real

Round fifteen's own finding — that this audit's rounds six through fourteen had one detection technique
and were structurally blind to a whole other bug class — got exactly the response it called for: a fresh
sweep organized by bug class across the *entire* `app/` tree, not a roadmap-row slice, and not reusing the
create/delete-sibling technique that had already been driven clean and independently reconfirmed. Seven
parallel agents, each assigned one bug class with no overlap: uncaught-exception gaps (the class round
fifteen found), stuck-job/non-terminal-state bugs, race conditions/concurrency, injection surfaces
(command/SQL/path), authorization/IDOR on both the account+API side and the WHM admin/reseller side, and
general logic errors (off-by-one, dead code, copy-paste bugs).

Two of the seven came back completely clean — a genuinely reassuring result, not a shallow one. Both
authorization/IDOR passes independently re-derived the entire permission model from current code (not
trusting any prior audit's verdict) and found nothing: every account/API controller scopes resources to
the authenticated account, every WHM/reseller action enforces the right granular permission, the
suspended-actor-still-acting class fixed back in audit #07 is still correctly enforced. Injection surfaces
came back clean too — no command injection (every `Process::run()` call uses the array form, no raw
`exec`/`shell_exec`/backticks anywhere), no SQL injection (the four raw-SQL call sites in the whole
codebase are all parameter-bound or fully static). One informational note fell out of the WHM-side pass:
it expected to find a literal "Terminal" command-runner controller and couldn't — checked against
`docs/roadmap.md`, this was a deliberate scope decision from early on, shipped as **Diagnostics** instead
(a real command-execution terminal was rejected as too high a compromise-blast-radius for a web app), not
a gap.

The other five passes found real, substantive bugs — confirming the methodology change was the right call,
not a formality. In severity order:

**cPanel import's tar extraction had no symlink guard — a real cross-tenant RCE.** Directly verified,
not just agent-reported: `CpanelImportService::validateAndExtract()` checked every archive member's
*name* (rejecting absolute paths, `../` segments, an allowlisted top-level directory) but never its
*type*. A crafted `.tar.gz` containing a symlink member (e.g. named `homedir` → another customer's
`public_html`) followed by an ordinary-looking regular-file member written through it lands outside the
staging directory entirely, on a completely different hosting account's live web root. Reachable from the
self-service, non-admin "Restore from cPanel Backup" upload — any customer could plant a webshell on
another customer's site. Every *other* tar extraction in this codebase already has this exact guard via
the privileged provisioning script's `validate_tar_archive_safe()`; this one PHP-side, unprivileged
extraction (which runs *before* any privileged script touches the archive) was simply never brought in
line. Fixed with a PHP-side equivalent: list the archive with `tar tvf` and reject any symlink or
hard-link member before ever calling `tar xzf`.

**Webmail's own pagination silently drops a message on every page turned.** `ImapMailboxClient::
listMessages()` over-fetched by one row (`limit($perPage + 1, $page)`) as a cheap trick to detect whether
a next page exists — but the underlying library (`webklex/php-imap`) reuses that same inflated count as
the page-size multiplier for its own offset math, not just the fetch count. Page 2 of a 100-message inbox
silently starts one message late, page 3 two messages late, permanently — no error, no warning, in the
panel's own flagship webmail client. Fixed by fetching exactly `$perPage` messages at the correct offset
and determining "has more" from a separate, cheap total-count check instead of the overfetch trick.

**Account backup/restore was the one privileged operation in this codebase that never went through a
queue — and had zero concurrency guard.** Every other long-running privileged action here is dispatched
as a queued job with an "already running" gate; backup/restore ran synchronously in the web request
instead, so two near-simultaneous requests for the same account (a double-click, two open tabs)
genuinely raced across two php-fpm workers. Worse, the backup destination path only had 1-second
resolution, so two clicks in the same second wrote into the *identical* directory — two independent
tar/mysqldump processes corrupting each other's output while the row still got marked `Completed`. Fixed
with a per-account `Cache::lock` (sized to each operation's real `Process` timeout, not a copy-pasted
number) around backup/restore/restorePath, plus a random path suffix as defense in depth against the
underlying collision even if the lock were ever bypassed.

**The exact bug class round fifteen found — uncaught `Process::timeout()` exceptions leaving a run
stuck in a non-terminal status forever — turned out to be systemic, not a one-off.** The dedicated sweep
found 9 more instances: MariaDB upgrade/snapshot/restore (whose stuck-Running gate covers all three
actions together, so a stuck upgrade could have blocked the admin's own recovery path — restoring the
pre-upgrade snapshot), the OS package manager, the panel's own self-update mechanism, PHP extension
toggling, system package updates, WAF engine install/ruleset-update, both cPanel/account-import paths
(new-account and into-existing-account, each also leaking its staging directory on throw), Git Deploy,
and the panel's own TLS certificate issuance. All fixed with the identical `try/catch(Throwable)`-mark-
Failed-rethrow shape established in round fifteen. `AccountProvisioningService::issueSsl()` got one
high-leverage fix instead of three: it's called from three separate jobs that already correctly branch on
its boolean return value, so catching the timeout inside `issueSsl()` itself and returning `false`
(matching its own existing "false on failure" contract) fixed all three downstream call sites — one of
which, `ConvertAddonDomainJob`, would otherwise have left `AddonDomainConversion.status` stuck at
`Running` forever even though the underlying account conversion had already fully succeeded — without
touching any of the three job files.

**A structural gap sits underneath all of the above and wasn't fixed this round: nothing protects
against a hard queue-worker kill.** A job's own declared timeout is enforced by the queue worker killing
the process via signal, which bypasses every `try/catch` no matter how well-placed — the same is true of
an OOM kill or a `systemctl restart` mid-job. Today exactly one of roughly eighteen "Run"-shaped models
in this codebase has a self-healing sweep for this (`SubscriberCampaign`, via the existing
`ResumeStuckSubscriberCampaigns` command). This is a real design decision — a generalized reconciliation
sweep, not a mechanical fix — and is being deliberately held for a follow-up round rather than built
unilaterally.

**Three more concurrency races, and a batch of "friendly error instead of raw 500" fixes, rounded out
the sweep.** `DnsZoneService::restoreDefaultRecords()`/`ensureMailAuthRecords()`/`importRecords()` had
the identical unlocked-check-then-insert race as the backup fix above — no DB unique constraint backs
`dns_records`, so a plain double-click on "Restore Default Records" could write duplicate SPF/DMARC/NS
records (a duplicate SPF TXT record causes a real RFC 7208 `permerror`, a silent deliverability
regression, not just a cosmetic dupe) — fixed with the same per-zone lock pattern. The Git Deploy webhook
endpoint had the exact race its sibling `deployNow()` button was already fixed for, just never applied to
the public, token-authenticated webhook path — worse there since GitHub/GitLab/Bitbucket routinely
deliver duplicate webhook events; now shares the same lock key so both serialize against each other.
Subscriber campaigns' manual-paste and CSV-upload recipient sources deduped independently of each other,
so the same email present in both got double-emailed and double-counted — now deduped across both
sources. And five services (`WebdavAccountService`, `SubdomainService`, `AppInstallerService`,
`FtpAccountService`, `DatabaseManagerService`'s `createDatabase()`/`createUser()`/`allowRemoteHost()`) had
a real unique DB constraint already backstopping a create-race, but surfaced the losing request's
`QueryException` as a raw 500 instead of a friendly error — now caught and converted, with
`allowRemoteHost()`'s catch additionally attempting a best-effort revoke of the just-added MySQL grant so
a lost race doesn't leave an orphaned, invisible grant behind.

Full suite now at 2,191 tests (up from 2,153), `pint --test` clean. Live-verified: deployed commit
confirmed live on `panel.dev.knj.network` via direct SSH check of `/srv/panel`'s HEAD against the pushed
commit, queue worker confirmed restarted fresh and running the new code.

```
round:    initial  6   7   8   9  10  11  12  13  14  15  16
findings:   many   4  ~30  9  13  13   5   2   2   0   2  ~20
```

Round sixteen's count jumping back up after round fourteen's zero isn't a regression in the codebase —
it's the audit finally looking somewhere it never had before. The next round returns to confirmation
mode against this fix set.

## Round sixteen — closing the held-open gap: a generalized stuck-run sweep

The one item deliberately held open above — nothing defends against a *hard* queue-worker kill (the
job's own timeout enforced via signal, an OOM kill, or `systemctl restart knjpanel-queue` landing
mid-job, which the deploy pipeline itself triggers on every push) — got built the same day. Every
try/catch fix above only closes the *throwing*-failure path; a hard kill bypasses all of them
regardless of placement, since the process is terminated before any PHP code, including a catch
block, can run.

Researched first rather than guessed: read every job class's own declared `$timeout` (`app/Jobs/*.php`
consistently declares one — `MariadbUpgradeRun`'s three jobs at 630s, `PanelUpdateRun`/`SystemUpdateRun`
at 1830s, `WafInstallRun` at up to 3900s, and so on) and every "Run"-shaped model's schema (all share
the same `status`/`started_at`/`completed_at` shape, backed by an enum whose `Running`/`Failed` cases
share the identical string values across every enum in this codebase — `SystemUpdateRunStatus`,
`WafInstallRunStatus`, `BackupStatus`), before designing the fix — a wrong threshold either fires on a
genuinely-still-running operation or leaves a truly stuck one undetected for too long.

New scheduled command, `knjpanel:resume-stuck-system-runs` (every 5 minutes, matching
`CheckServerHealth`/`CheckWafHealth`'s own cadence), generalizes the one prior precedent for this
class of problem — `ResumeStuckSubscriberCampaigns`, which *re-dispatches* a fresh job because a
campaign send is safely resumable (it only ever acts on rows still `queued`) — to a dozen other models
that aren't safely resumable the same way: MariaDB upgrade/snapshot/restore, panel self-update, system
package updates, WAF engine install/ruleset-update, OS package install, PHP extension toggle, Git
Deploy, both cPanel/account import paths (with their staging directories cleaned up too, closing the
same leak round fifteen's uncaught-exception fixes already closed for the *throwing* path), and account
backup/restore. None of these can just be resumed from wherever they left off, so the sweep's job is
narrower: mark the stuck row `Failed` — exactly what its own try/catch would already have done had the
process lived long enough to reach it — clearing the "already running" gate every one of these
controllers checks, and alert an admin via the existing `AdminAlertService`/webhook channel (a new
`job.stuck` event, a new `notify_on_stuck_job` toggle on the Notifications page, following the exact
pattern already used for `waf.degraded`/`backup.failed`/`server.unreachable`) so a stuck operation
doesn't go unnoticed the way every one of these did before this sweep existed.

**`AppInstallation` is deliberately excluded**, and the reasoning is worth recording since it's the one
place this sweep intentionally does *less* than it could: every other model here can be blind-flipped
to `Failed` safely because that's exactly what its own try/catch already does on a normal failure.
`AppInstallerService::runInstall()` is different — on a normal failure it doesn't just flip a status,
it *rolls back* (deletes the database user and database it created, tears down what it wrote). A hard
kill mid-install could leave those same resources half-created with no rollback ever having run; a
blind status flip here would leave them permanently orphaned and invisible, worse than leaving the row
alone. That needs bespoke, install-aware reconciliation this sweep doesn't attempt — tracked
separately, not silently skipped.

Six new tests (`ResumeStuckSystemRunsTest`) cover: a genuinely stuck run being marked Failed with an
admin alert sent, a run still within its own timeout being left alone (no false positives), staging-
directory cleanup actually running, a second model type (Git Deploy) confirming the sweep generalizes
correctly rather than being hardcoded to one shape, `AppInstallation` provably being left untouched, and
no alert firing when nothing is stuck. Full suite now at 2,197 tests (up from 2,191), `pint --test`
clean. Live-verified: deployed commit confirmed live via SSH, and `php artisan schedule:list` on
`panel.dev.knj.network` confirms the new command is correctly registered and due to run within 5
minutes, alongside the existing subscriber-campaign sweep it was modeled on.

This closes every item raised by round fifteen's own methodology critique. The codebase is not being
declared "launch-ready" from this alone — round fifteen also called for confirmation rounds against
each fix set, which continue from here.

## Round seventeen — confirming the bug-class sweep, and its own blind spot

The first confirmation pass against round sixteen's methodology change: the same seven bug classes,
each re-derived fresh from current code rather than trusting round sixteen's verdicts. Both
authorization/IDOR passes came back clean again, independently re-confirming the permission model with
no new gaps. The other five classes found real issues — some new, one a gap in round sixteen's own new
code, and one that exposed a structural limit in the technique itself.

**The same "already running" check-then-act race round sixteen fixed once, in `GitDeployController`,
had never been propagated to the 9 other controllers with the identical shape.** Every one of them
follows the same pattern — check whether a run is already in progress, create the DB row, dispatch the
job — with no lock between the check and the create, so two concurrent requests can both pass the
check and both dispatch: `DatabaseUpgradeController` (snapshot/apply/restore, one shared lock, since
the three are mutually exclusive on the same database), `WafController`, `SystemUpdateController` (two
independent locks — panel updates and OS package updates don't need to exclude each other),
`PackageManagerController`, `PerlModuleController`, `PhpExtensionController` on the WHM side, and
`CpanelImportController`, `PhpPearPackagesController`, `PerlModulesController` on the account side.
`PerlModuleController`/`PerlModulesController` deliberately share one lock key across the WHM/account
boundary, since both write to the same global Perl module state. All nine wrapped in
`Cache::lock()->block(5, ...)`, lock timeout left uncaught — matching the precedent
`GitDeployWebhookController` already established for exactly this failure mode.

**A cross-table duplicate race with zero DB backstop**: `mailboxes`, `mail_forwarders`, and
`mailing_lists` share one namespace (a domain's local part can only mean one thing), but nothing
enforced uniqueness *across* those three tables, only within each one. Two concurrent requests — a
mailbox create and a mailing list create for the same address, or either against itself — could both
succeed, leaving `admin@example.com` simultaneously a mailbox and a mailing list with undefined mail
routing. `MailboxService::createMailbox()` and `MailingListService::createList()` now share one
`Cache::lock("mail-local-part:{$site->id}:{$localPart}")` key, serializing the two against each other,
plus a `QueryException`-to-friendly-error catch on the same-type case. `SubscriberService::subscribe()`
got a `QueryException` retry rather than an error, since a double-submitted public subscribe form isn't
really a failure — whichever request won already satisfies the caller's intent.

**The 4-service mail auth-map family, and 6 more `sync()`-style methods, rebuilt a shared file from
every DB row with no lock against a second concurrent trigger.** `MailAuthMapService`,
`ForwarderAuthMapService`, `MailingListAuthMapService`, and `DefaultAddressAuthMapService` each
regenerate their own file from scratch on every call; two overlapping regenerations risk a truncated or
interleaved write, install-wide. Each got its own global lock, timeout left uncaught (matching this
codebase's existing convention for from-scratch rebuilds). `IpBlockerService`, `CronJobService`,
`WebdavAccountService`, `DirectoryPrivacyService`, `DirectoryIndexService`, and
`DatabaseManagerService::syncRemoteAccessFirewall()` had the identical shape, one lock each. Three of
these — `WebdavAccountService::create()`, `DirectoryPrivacyService::protect()`,
`DatabaseManagerService`'s remote-access methods — call `sync()` from inside their own
`try/catch(RuntimeException)` rollback block (not a `DB::transaction()`), so a raw, uncaught
`LockTimeoutException` there would silently bypass that rollback and orphan rows/files/grants; those
six convert the timeout to a friendly `RuntimeException` instead of leaving it raw, a deliberate,
documented departure from the auth-map family's own convention right above it, made for that specific
reason.

**`servers.ip_address` had no DB constraint at all.** Two admins completing a server-link at the same
moment could register two `Server` rows under the identical IP with no error from either request.
Added a nullable-safe unique constraint, and wrapped `ServerController::completeLink()`'s create in a
`QueryException` catch that returns a friendly error instead of a raw 500 to whichever request lost the
race.

**CalDAV/CardDAV's sync-collection `REPORT` could hand a client one row past the limit it asked for.**
`AddressBookBackend`/`CalendarBackend::getChanges()` queried `limit() + 1` — dead code left over from a
cursor/has-more pattern that was never actually wired up — and returned every row fetched, not just the
requested count. Fixed by querying exactly the limit.

**Round sixteen's own new sweep had a gap in its own model list.** `ResumeStuckSystemRuns`' 11-model
`CONFIGS` array was missing `CpanelTransferRun`, a same-shaped Run model that existed before round
sixteen shipped — added with the job's own timeout (3630s) plus the standard 120s buffer.
`AddonDomainConversion` needed different treatment entirely: unlike every other model in the sweep, its
remaining provisioning work (the `convert-addon-domain` script action) can't be safely re-run after a
hard kill — its own guard clauses fail loudly on a second attempt, and the DB-side account/site
re-parenting it depends on has no idempotent check either. A stuck conversion is now left completely
untouched (no status flip) and raises a distinct, more urgent admin alert explicitly stating nothing
was auto-changed, rather than either silently re-attempting unsafe work or blind-flipping a status that
would hide a real half-finished conversion. The reused `job.stuck` webhook event's label was updated to
cover both the auto-fail case and this alert-only case, since one event name now means either.

**What this round's own findings say about the technique itself.** The create/delete-sibling and
persist-before-verify techniques that closed out rounds six through fourteen, and the uncaught-exception
technique round fifteen added, are both structurally blind to this round's biggest finding — the same
race pattern copy-pasted across nine controllers is a *propagation* failure, not a shape a single-file
review would catch, and only surfaced because this round's race-conditions pass re-derived every
controller from scratch rather than checking whether the one already-fixed instance had a sibling.
Round sixteen's resume-sweep, in turn, only defends against a *hard* kill bypassing every try/catch;
this round found the sweep itself needed two more model entries, the same "was this fix actually
propagated everywhere it should have been" question applied recursively to round sixteen's own new
code. No further methodology gap is visible from here, but that was also true, at the time, after round
fourteen's first clean pass.

Full suite now at 2,230 tests (up from 2,197), `pint --test` clean. Live-verified: deployed commit
confirmed live via SSH (`/srv/panel` HEAD matches the pushed commit), the new
`servers_ip_address_unique` index confirmed present via a direct `SHOW INDEX` query, the
`2026_08_26_090000_add_unique_constraint_to_servers_ip_address` migration confirmed `Ran`, and the
queue worker confirmed restarted fresh by auto-deploy.

```
round:    initial  6   7   8   9  10  11  12  13  14  15  16  17
findings:   many   4  ~30  9  13  13   5   2   2   0   2  ~20  ~15
```

A further confirmation round may follow if requested, same standing discipline as every round above.

## Round eighteen — closing the propagation gaps round seventeen found

A short, direct follow-up: round seventeen's own 15 findings, fixed and verified. The 9 controllers
with the copy-pasted "already running" race each got the same `Cache::lock()->block(5, ...)` treatment
already applied to `GitDeployController`; the mailbox/mailing-list local-part race got the shared
`"mail-local-part:{$site->id}:{$localPart}"` lock plus `QueryException`-to-friendly-error handling on
both sides; the 4-service mail auth-map family and 6 more `sync()`-style methods each got their own
lock, six of them converting `LockTimeoutException` to a friendly `RuntimeException` since their
callers roll back via `try/catch(RuntimeException)` rather than `DB::transaction()`; the
`servers.ip_address` gap got a real unique constraint plus a matching catch on the create race; and the
CalDAV/CardDAV pagination bug got the one-line fix. Two agents split the fixes with no overlap. Full
suite at 2,197 tests, `pint --test` clean, deployed and confirmed live via SSH.

With round seventeen's fix set closed out, the plan was to run a confirmation pass and, if that came
back clean, cut a release — the same rhythm every prior methodology change has followed.

## Rounds nineteen and twenty — "no, do it properly," and it was right to ask

Before that confirmation pass ran, the question of whether to cut got raised directly, and the honest
answer was: no methodology used so far had actually gone two consecutive rounds clean on a *newly
introduced* technique — round fourteen's clean pass was the old technique fully exhausted, not this
one. Two things were offered instead of an immediate cut: a **targeted mechanical sweep** re-checking
the two exact patterns round seventeen had just fixed (not a fresh open-ended review) across every call
site in the codebase, and — if that also found more — a genuine methodology upgrade rather than another
repeat of the same fresh-eyes format.

**The targeted sweep immediately proved itself.** The check-then-act lock pattern came back completely
clean — every site already fixed, backed by a real DB constraint, or genuinely idempotent under
re-dispatch. The sync()-rebuild lock pattern did not: **9 more instances**, including two with real
operational weight. `DnsZoneService::writeZone()` itself — the method three of its own sibling methods
already wrapped in a lock — was unlocked when called directly by half a dozen other controllers/
services. And `MailServerConfigService`/`MailServerSettingsService`'s override-merge `apply()` had a
genuine lost-update race: two admins saving different fields on different mail-settings pages within
the same window could each read a stale value for the other's just-persisted field and silently drop
it from the live Postfix/Dovecot config on write — the widest blast radius of anything this sweep found,
reachable from at least five independent call sites across the mail-configuration surface.

**That result was reported honestly, together with what it meant: the two-fresh-audits-in-a-row
pattern this project had leaned on for confidence had a real blind spot the whole time.** Every prior
round used fresh-eyes review — read the code, use judgment, flag what looks wrong. That's good at
finding a bug the first time; it's unreliable at the mechanical task of checking "was this exact fix
propagated everywhere it should have been," because that's a cross-referencing exercise, not a
judgment call. The targeted sweep worked specifically because it stopped relying on judgment and
instead grepped for one exact shape and checked every call site against it, one by one — a different,
more reliable technique for that specific kind of question.

**The response was to scale that technique up, not just fix the 9 findings and stop.** Six parallel
mechanical sweeps ran across every major bug pattern this whole audit has ever found — not sampling,
not fresh judgment, but exhaustively grepping every call site of a given shape and checking each one's
current code, cross-referencing nothing against "should already be fixed" assumptions:

- **Persist-before-verify**, split three ways (mail/DNS/DB services, security/files/software services,
  every controller) — the technique that closed out rounds six through fourteen, re-run mechanically
  rather than via the create/delete-sibling comparison, specifically because that comparison depends on
  a reviewer noticing the sibling exists.
- **Uncaught Process-timeout / stuck-run model-list completeness** — the class round fifteen and
  sixteen's resume-sweep addressed, re-verified against every `Process::timeout()` call site and every
  Run-shaped model in `app/Models/`.
- **DB-constraint-backed create-race completeness** — every `::create()` call against a model with a
  real unique constraint, checked for a matching `QueryException` catch, plus a check for models that
  plausibly need a constraint they don't have.
- **IDOR/authorization**, a third independent pass using a different technique than the two prior
  fresh-review passes: systematic cross-referencing of every mutating action against its siblings
  across account/API/WHM, rather than reading each controller in isolation.

**Two of the six came back genuinely clean** — uncaught-timeout/stuck-run completeness (every
`Process::timeout()` call site already correctly wrapped, the `ResumeStuckSystemRuns` model list
already complete against every Run-shaped model in `app/Models/`) and IDOR/authorization (a third
independent pass, ~450 mutating actions cross-checked, two near-misses investigated and correctly
ruled out rather than hand-waved past). **The other four found real, substantial issues** — roughly 68
distinct findings before deduplication, collapsing to about 45 after tracing shared root causes (the
`LockTimeoutException`/`RuntimeException` mismatch alone explained 9 separate controller-level
symptoms once traced back to `DnsZoneService`).

In severity order, the notable ones:

**`WebdavAccountService::changePassword()` was completely silent on failure — the most severe finding
of the whole sweep.** It persisted the new password hash to the DB, then called `writeHtpasswd()`,
which called a bare `file_put_contents()` with no return-value check at all — unlike this exact file's
own `create()`/`delete()`, which both carefully guard this. A disk-full or permissions failure meant
the DB showed the new password as active, the UI said "Password updated," and the real htpasswd file
nginx actually authenticates against silently kept the old password — with zero error signal anywhere.
Worse than a raw 500, since nothing at all indicated anything had gone wrong. Fixed to check the write
and roll the DB hash back to the previous value on a confirmed failure, mirroring `create()`'s own
rollback shape.

**The `LockTimeoutException`/`RuntimeException` mismatch was systemic, not a one-off.** `DnsZoneService`'s
four locked public methods left the exception type uncaught on the theory that `DB::transaction()`
rollback happens regardless of whether it's caught — true for data integrity, but every one of
`Account/DnsController`, `DynamicDnsController`, `Api/DnsRecordController`, the public Dynamic DNS
update endpoint, and `Whm/DeliverabilityController` already had `catch (RuntimeException $e)` written
specifically to show a friendly message on exactly this failure mode — dead code that looked handled
but wasn't. A separate instance of the same class of gap turned up in `GitDeployWebhookController`
itself, which round seventeen had treated as the established "leave it uncaught" reference — the
precedent nine other fixes were built on turned out to rest on an existing, unnoticed bug, not a
deliberate design choice. All now convert to a friendly `RuntimeException`, and both the webhook
controller and its account-side sibling `GitDeployController::deployNow()` now give the same graceful
"already running" response on genuine lock contention that their fast-path check already gave for the
common case.

**~30 more create-race sites had a real unique constraint but no friendly-error handling** — the exact
same mechanical fix already applied to `WebdavAccountService`/`ServerController` last round, just never
propagated to `AccountProvisioningService`'s `createAccount()`/`convertAddonDomainToAccount()`,
`AccountCollaboratorService`, `DomainsController`, `IpBlockerService`, `DirectoryPrivacyService`,
`DirectoryIndexService`, `MailingListService`, `WebmailContactsController`, `SubscriberService`'s CSV
import path, `FirewallService`, `DynamicDnsService`, both CalDAV/CardDAV backends (which needed
`Sabre\DAV\Exception\Conflict` instead of a generic exception, to keep the DAV protocol response
intact), `PhpVersionController`, and `ResellerController`. `servers.hostname` also had no constraint at
all, alongside `servers.ip_address`'s from last round — added, with a matching pre-check on
`ServerController::completeLink()` and the three server-bootstrapping console commands.

**~15 more missing-catch and persist-before-verify bugs**, all found by the controller-sweep's
systematic sibling comparison: `Whm\AccountController`'s `store()`/`updatePassword()`/`destroy()`/
`bulkDestroy()` all lacked the try/catch their own sibling `suspend()`/`unsuspend()` already had (the
same file, the exact comparison technique that closed rounds six through fourteen, just never run
mechanically against every method in every file); `AccountImportController`/`CpanelImportController`/
`AddonDomainConversionController::store()` had the identical gap; `NginxSettingsController`/
`PhpSettingsController` persisted every Setting before confirming `apply()` succeeded — and a naive
reorder would have made it wrong in a *different* way, since both services read `Setting::get()`
directly rather than taking new values as a parameter, so both got refactored to take an explicit
`$data` array instead, with `WafService`/`InstallWafEngineJob`/`PanelCertService` (re-apply-from-
current-state callers) updated to a new `NginxSettingsService::currentSettings()` helper;
`MailSettingsController::updateServer()`'s `mail.active_server_id` switch had the same stale-read
exposure as the mail-settings lost-update race above, now going through the same lock;
`SecurityController::unban()`/`FirewallService::unban()` called `fail2ban-client` completely
unwrapped; `BackupHistoryController::destroy()`, `Api\AccountController::suspend()`/`unsuspend()`,
`DnsOnlyZoneSyncController::sync()`, and `DnsReplicationController::syncNow()`'s unprotected IP
fallback all got the missing catch their siblings already had; `SystemPackageUpdateService::
fetchPendingUpdates()` got the same `ProcessTimedOutException` catch its own sibling `apply()` already
documents needing, plus a reorder so a failed check no longer overwrites the last-checked timestamp
with a false-clean state; `BackupService::delete()` now checks the real `rm -rf` result before dropping
the DB row, instead of silently orphaning the files on a failed removal; `MariadbUpgradeService::
runStreaming()` no longer persists a `snapshot_path` that looks usable on a run that ultimately failed;
and `SubscriberService::subscribe()`/`confirm()` now log-and-continue on a synchronous mail-send
failure rather than 500ing after the real subscription state change already succeeded, while the
sibling `SubscriberCampaignController::sendTest()` — which has no real state worth preserving — gets
the opposite fix, a proper error surfaced instead of silent success.

Seven fix agents split the ~45 findings with no overlap (one accidental double-assignment — both
independently reached the identical correct fix for `DnsClusterService`, detected the collision, and
one deferred to the other rather than duplicating). Two items were deliberately **not** blindly fixed:
`AccountProvisioningService::removeAccount()` deletes an account's DB rows before the real system-level
teardown confirms success, and `convertAddonDomainToAccount()` commits the DB reparenting before the
actual conversion job runs, with no revert on failure — both real gaps, but account deletion/conversion
is high-blast-radius enough to warrant a deliberately-designed fix in a dedicated round rather than a
rushed one under this sweep's own momentum.

Full suite now at 2,302 tests (up from 2,197), `pint --test` clean across the whole repository.
Live-verified: deployed commit confirmed live via SSH, both new unique-constraint migrations
(`servers.ip_address` from last round, `servers.hostname` from this one) confirmed `Ran`, queue worker
confirmed restarted fresh.

```
round:    initial  6   7   8   9  10  11  12  13  14  15  16  17  18  19
findings:   many   4  ~30  9  13  13   5   2   2   0   2  ~20  ~15  15  ~45
```

**What this pair of rounds actually settled.** The count going back up after round eighteen's short,
clean-looking fix pass isn't the methodology failing — it's confirmation that "the last round's fixes
looked complete" is not the same claim as "the last round's fixes *were* complete," and that the gap
between those two claims is exactly what a mechanical, exhaustive, cross-referencing sweep catches and
a fresh-eyes review — no matter how many times repeated — structurally cannot. Two techniques (IDOR,
uncaught-timeout) that had already gone through fresh-eyes review twice were re-checked a third time,
this time mechanically, and came back genuinely clean both times — that's real signal, not noise, in a
way that "round fourteen was clean" alone couldn't fully support in hindsight. The standing plan going
forward: any future confirmation round uses this mechanical, every-call-site technique by default, not
the fresh-eyes format rounds six through seventeen relied on — fresh-eyes finds a bug once; only
exhaustive cross-referencing confirms a fix actually landed everywhere it needed to.

## Round twenty — a second morning's mechanical confirmation pass

A fresh day, the same eight-way mechanical sweep from round nineteen, re-run against round nineteen's
own fixes with the explicit goal of finding out whether the codebase could go two consecutive rounds
clean on this technique for the first time. It could not — but the shape of what it found, and what it
didn't, is itself the useful result.

**Two of the eight came back clean a third time in a row**: uncaught-timeout/stuck-run completeness
(every `Process::timeout()` call site still correctly wrapped, `ResumeStuckSystemRuns`'s model list
still complete) and check-then-act lock races (every site still locked, DB-constrained, or genuinely
idempotent) — though the latter found two NEW instances of a *different* pattern in the process (see
below). **IDOR/authorization came back clean a fourth time**, this pass done by an agent that caught
and openly corrected its own mistake mid-task: it initially reported two false-positive findings from
one of its own sub-agents (`Whm\BackupHistoryController`, `Whm\DatabaseAccessHostsController`,
claimed reachable by any Reseller with zero ownership check) without verifying them, then went back,
traced `routes/web.php` itself, found both routes sit inside an `admin.access`-gated block that blocks
Resellers outright before the controller ever runs, and retracted the false claim before it reached a
fix queue. 46 models, 166 real call sites checked, 0 confirmed cross-tenant IDOR.

**The other five found real issues** — smaller in volume than round nineteen's ~45 (expected, since
round nineteen had already closed the bulk of what existed), but genuinely new, not leftovers:

**Two more concurrency races with zero lock, not caught by round nineteen's sweep because they're
outside the files that sweep happened to check.** `AccountProvisioningService::
convertAddonDomainToAccount()` had no lock at all — two near-simultaneous conversions of the same addon
domain could both pass the `is_primary` check and each create a separate new account for it, worse than
any of round nineteen's controller-level create-race findings since this one has no DB constraint
backing it either. `BackupService::backupPanelDatabase()` never got the same lock + randomized
destination-path suffix its sibling `backupAccount()` already carries from an earlier round's fix — the
exact "two concurrent backups collide on one path" bug that sibling was hardened against, still open on
the panel-database side.

**A confirmation of round eighteen's own systemic finding, this time in a file nobody had swept yet.**
`GreylistExemptionService::regenerate()` rebuilds the entire shared postgrey whitelist from every exempt
account's domains, with no lock — the identical shape already fixed for the mail auth-map family two
rounds ago, just never propagated into this specific service. Same fix, same reasoning, new file.

**A genuine caller-side gap traced back to a design decision made two commits into this exact round of
fixing.** `Whm\MailSettingsController::updateServer()`'s single catch block only actually covered 2 of
the 6 calls it wraps — the 4 auth-map `regenerate()` calls it also makes deliberately leave
`LockTimeoutException` uncaught, by design, because their *other* callers roll back via
`DB::transaction()` instead. A lock timeout on any of those 4 inside `updateServer()` specifically
skipped the `mail.active_server_id` revert entirely, silently pointing the panel at a mail server whose
config sync never actually completed. Fixed at the call site with a second catch clause, without
touching the 4 services' own convention (which is correct for their other callers).

**The `LockTimeoutException`-uncaught pattern round nineteen's own agents disagreed about turned out to
be real, confirmed independently three separate times.** Round nineteen's WHM-controller sweep flagged
`PackageManagerController`, `PerlModuleController`, and `PhpExtensionController`'s install-run actions
for this, but a *different* agent in the same round argued the identical shape across ten files was "a
deliberate, consistent, codebase-wide convention, not a bug" — a genuine disagreement left unresolved at
the time. Round twenty's confirmation sweep independently re-flagged all three, *plus* the exact same
gap in three more account-side siblings (`CpanelImportController`, `PerlModulesController`,
`PhpPearPackagesController`) that share the identical `Cache::lock()->block(5, ...)` shape. Three
independent verdicts against one dissent settled it: fixed all six, plus a missing duplicate-install
guard on `PhpExtensionController::installIonCube()` that had no protection at all, unlike its sibling
`toggle()` in the same file. While fixing these, 6 existing tests turned out to have literally asserted
the old raw-500 as the *correct* behavior (`assertServerError()`) — the same "test encodes the bug"
pattern this audit keeps finding, now in test files written to cover a fix from an earlier round.

**Five more uncaught `Process`-timeout gaps, the exact class rounds fifteen and sixteen first found,
still not fully swept.** `DnsController::refreshDs()` (every sibling DNSSEC action in the same file
already catches this), `SystemRebootController::store()`, `TrackDnsController::index()`,
`WafHealthCheckService::check()` (fixed to return its own existing "degraded" contract instead of
throwing, so no controller change was needed), and `WebdavSettingsController::update()`. Same shape,
same fix, five more sites a prior sweep's grep pattern didn't happen to catch.

**Seven more sibling-inconsistency missing-catch bugs**, found the same way rounds six through fourteen
found their originals — comparing a fixed method against its unfixed neighbor in the same file:
`SslController::retry()`, `DirectoryPrivacyController::removeUser()` (its sibling `addUser()` already
caught this), `DatabaseBrowserController::show()`/`table()` (sibling `runQuery()` already did),
`SubscriberCampaignController::destroy()`, `TeamAccessController::resend()`/`update()`,
`WebmailController::downloadAttachment()`, and `MailEncryptionController::destroy()`.

**A real security finding — not a data-integrity bug, a permission bypass — reported by a peer Claude
session working the same codebase in parallel, cross-verified and folded into this round's fix wave.**
`Api\AccountController`'s six mutating actions (suspend/unsuspend/store/destroy/updatePackage/
updatePassword) checked `tokenCan()` — an ability a reseller can self-select when minting their own API
token — but never `hasResellerPermission()`, the admin-controlled, independently-revocable restriction
every equivalent `Whm\AccountController` action already enforces. A reseller whose `accounts.terminate`
permission was revoked by an admin could still terminate their own account through the API, because the
revoked permission was never consulted on that path — the web UI correctly blocked it, the API silently
didn't. `Api\DomainController::convert()` had the identical gap for `accounts.create`, and it's the
same bug commit that fixed `Whm\AddonDomainConversionController` for this exact issue two rounds ago —
fixed on the web side, never propagated to its own API sibling despite calling the identical
`AccountProvisioningService::convertAddonDomainToAccount()` underneath. Both now check the same
permission key as their web-side equivalent. The peer session's own audit process is worth noting: it
caught and explicitly retracted a fabricated interim result partway through ("I stated results for two
sub-agents that hadn't actually returned — that was wrong, redoing it for real"), then separately caught
two false-positive findings from its own sub-agent before they were reported as confirmed — the same
verify-before-trusting discipline this whole session has tried to hold itself to. Regression tests
specifically assert *denial* — a self-granted matching token ability does not bypass a revoked
`reseller_permission` — not just that the code compiles.

**Three more create-race sites the mechanical grep itself structurally couldn't see.**
`ChallengeResponseService::approve()` builds its row with `->make()` + a bare `->save()` rather than
`::create()`, so neither of the two grep patterns used to find every create-race site in round nineteen
matched it at all — found only because this round's agent read the file directly rather than trusting
the grep's completeness. `FtpAccountService::restore()`'s `updateOrCreate()` lookup key doesn't actually
match the real DB constraint shape (scoped by `account_id` in the lookup, but `username` alone is what's
globally unique), a subtle guard/constraint mismatch rather than a missing guard. `CreateInitialAdmin`
never got the same-round-added catch its three sibling bootstrap commands did.

Six fix agents split the ~24 confirmed findings with no overlap. Full suite now at 2,343 tests (up
from 2,302), `pint --test` clean. Live-verified: deployed commit confirmed live via SSH, VERSION and
queue-worker restart confirmed on `panel.dev.knj.network`.

```
round:    initial  6   7   8   9  10  11  12  13  14  15  16  17  18  19  20
findings:   many   4  ~30  9  13  13   5   2   2   0   2  ~20  ~15  15  ~45  ~24
```

**Where this leaves the "is it clean" question the user actually asked.** Not clean — two more rounds
in a row have now found real bugs, and there is no basis to claim a third round wouldn't find more.
What has changed, concretely, across rounds nineteen and twenty: three of eight mechanical sweep
categories (uncaught-timeout/stuck-run, check-then-act races, IDOR/authorization) have now independently
come back clean multiple times in a row using a technique that has proven, repeatedly, that it does find
real bugs when they exist — that's a meaningfully different claim than "nobody happened to look." The
remaining categories keep finding smaller, more scattered instances of patterns already identified and
already fixed elsewhere — sibling files that didn't get the same fix, grep patterns that structurally
couldn't see a `->save()` instead of a `::create()`, a caller with a narrower catch than the service it
wraps. That is a genuinely different failure mode than round nineteen's ~45 findings, which included the
first-ever instance of several bug classes (the mail-settings lost-update race, the silent WebDAV
password-write failure) rather than propagation gaps of already-known ones. Round twenty found zero new
bug *classes* — every single finding this round was the same shape as something already fixed in an
earlier round, just in one more file. That's real progress on breadth even though it isn't yet a clean
result. The account-deletion/conversion rollback design gap flagged and deliberately deferred at the end
of round nineteen is still open and still deferred — nothing in round twenty changed the case for
tackling that separately and deliberately rather than folding it into a sweep's momentum.

## Round twenty-one — the biggest sweep yet, run fresh the next morning

Same eight mechanical categories, same instruction: re-derive everything from the current code, don't
trust a prior round's "clean" verdict. This round found more than any single round before it — roughly
65 real findings — and, unlike round twenty, it wasn't just propagation gaps of already-known shapes.
Several genuinely new patterns turned up.

**The single most consequential finding: `ProvisioningScriptRunner::runLocal()` broke its own "never
throws" contract.** This class's own docblock describes itself as the one choke point every
mail-touching service calls into the privileged provisioning script through, and promises callers it
always returns a `ProvisioningResult` — never an exception. But its `Process::timeout(30)->run(...)`
call had no catch for a genuine timeout, so `ProcessTimedOutException` broke that promise silently for
every one of its ~19 dependent services (mail relay, mail filters, spam filtering, greylisting,
challenge-response, SMTP restrictions, mailing lists, all four mail auth-map families, mailbox
create/delete/password, ClamAV, DKIM). None of those 19 services wrap their own calls into the runner in
a try/catch, because the whole point of the runner is that they shouldn't have to. Fixed at the one
source, not at 19 call sites.

**A second systemic root cause, closing a gap this session has referenced but never directly fixed:
`LockTimeoutException` extends the base `Exception`, not `RuntimeException`, and it was silently
escaping through all four mail auth-map `regenerate()` methods** (`MailAuthMapService`,
`ForwarderAuthMapService`, `MailingListAuthMapService`, and `DefaultAddressAuthMapService`, the last one
caught only after this round's mail-fix agent flagged it as "found but out of scope" and I fixed it
directly). Eleven separate controller actions across `MailController` and `MailingListController` only
`catch (RuntimeException $e)`, which never actually caught it — a raw 500 under concurrent mail writes.
Converting the exception at the four source methods fixes all eleven call sites at once, the same
"fix the contract, not the callers" shape as the provisioning-runner fix above.

**A third systemic root cause, and this round's own new construction-shape lesson: `updateOrCreate()`
is treated throughout this codebase as if it were atomic, but it isn't** — it's Laravel's own
select-then-write, exactly as racy as the bare `::create()` calls prior rounds already fixed. Fourteen
more unprotected `updateOrCreate()`/`new Model()+save()` sites turned up this round (MIME types, hotlink
protection, website optimization, mailbox certificates, database grants, error pages, default address,
quick sites, autoresponders, server linking, the known-IP recorder, and the panel's own settings store),
two of them on **public,
unauthenticated** unsubscribe links, and one — the known-IP recorder — firing on **every login**, where
a losing race is now swallowed silently rather than surfaced as an error, since the desired end state
(the IP is recorded) is already true either way.

**Two more prior-round regression tests turned out to have encoded the bug they were meant to catch.**
`WafControllerTest`'s lock-contention test asserted `assertServerError()` — a raw 500 — as the correct
outcome for a near-simultaneous ruleset update, and `SystemUpdateControllerTest`/
`DatabaseUpgradeControllerTest` had the identical pattern. Same "test encodes the bug" failure mode
round twenty already found once, now confirmed as a recurring risk of writing a regression test at the
same moment a bug is discovered rather than after it's actually fixed — both updated in place alongside
their fixes.

**Genuinely new bug shapes, not propagation gaps:** `PanelCertService::issue()` could leave
`panel.ssl_status` reading "active" in the DB after a successful cert issuance if the trailing nginx
config apply then failed, with no admin-visible signal beyond a log line nobody was watching — now
reverts the status and fires an admin alert through the same channel per-account SSL failures already
use. `SystemCronJobService::updateNotifyEmail()` committed the new notify-email setting before calling
`sync()`, with no rollback on failure — a direct regression against `CronJobService::updateCronEmail()`
in the very same codebase, which wraps the identical shape in a transaction specifically to prevent
this. `ChallengeResponseController::approve()` unconditionally told the user their held mail had been
released, even when the release script actually failed and nothing moved. `TeamAccessController::resend()`
rotated the invite token before attempting to send the new invite email, so a failed send left the
collaborator with neither a working old link nor a working new one — the send now has to succeed before
the rotation commits. `DatabaseManagerService::allowRemoteHost()`'s failure-rollback branch deleted the
DB row on a firewall-sync failure but never revoked the just-created MySQL grant itself, unlike its own
`QueryException` branch two lines above, which does — a live, fully-privileged remote MySQL account that
would have been invisible to the panel from then on.

**IDOR/authorization came back clean for the fourth round running** — 151 files, ~450 traced actions,
zero new findings, the round-20 API reseller-permission fix confirmed still intact and its pattern
holding everywhere else a reseller can reach another party's resource. That's the strongest "this
category is actually solid" signal this audit has produced yet.

Fourteen fix agents split the ~65 confirmed findings by file ownership to avoid conflicts (all sharing
one working tree, not isolated worktrees — several agents independently noted seeing each other's
in-progress changes via `git status`, which is expected and harmless since Pint's `--dirty` formatting
pass is idempotent). Two findings fell through the cracks of that split — `DiagnosticController`'s
identical uncaught-timeout shape and the `DefaultAddressAuthMapService` sibling gap mentioned above — and
were fixed directly rather than spinning up more agents for two single-file changes. Full suite now at
2,429 tests (up from 2,343), `pint --test` clean across the whole codebase. Live-verified: deployed
commit confirmed via SSH, VERSION and queue-worker restart confirmed on `panel-dev.knj.network`.

```
round:    initial  6   7   8   9  10  11  12  13  14  15  16  17  18  19  20  21
findings:   many   4  ~30  9  13  13   5   2   2   0   2  ~20  ~15  15  ~45  ~24  ~65
```

**Where this leaves the "is it clean" question.** Not clean, and this round is the clearest evidence
yet that "clean" was never going to arrive on a fixed schedule — round twenty-one found more real bugs
than any prior round, including round nineteen's original ~45. But the *character* of what it found
matters as much as the count. Three of the biggest finding clusters this round (the provisioning-runner
contract break, the auth-map `LockTimeoutException` leak, the `updateOrCreate()` atomicity assumption)
are each a single root-cause fix that closed somewhere between 11 and 19 downstream symptoms at once —
this is a different shape of progress than round twenty's one-bug-one-file propagation gaps, and a
different shape again from round nineteen's scattered first-instances. IDOR/authorization's fourth
consecutive clean round is the strongest single data point in the user's favor: it's the category where
a real bug would matter most (cross-account data exposure), and it's the category that's held up best
under the most scrutiny. The account-deletion/conversion rollback design gap flagged at the end of round
nineteen, and still open through rounds twenty and twenty-one, remains the one deliberately-deferred item
this audit keeps carrying forward rather than folding into sweep momentum.

## Closing the deferred account-deletion gap — a deliberate design pass, not a sweep fix

Flagged at the end of round nineteen and carried forward, untouched, through rounds twenty and
twenty-one specifically because it needed real design rather than the mechanical pattern-matching
that closes everything else this audit finds. `AccountProvisioningService::removeAccount()` hard-deleted
the `Account`/`User` DB rows synchronously, then dispatched `DeprovisionAccountJob` to do the real
system-level teardown — deleting the Linux user, home directory, mail, databases — asynchronously,
best-effort, with every step wrapped in a log-and-continue `try/catch`. **The actual bug underneath
this was worse than the design gap alone suggested**: `deprovisionRaw()`, the one call that runs the
privileged `destroy` script, never checked its own script's exit code at all — it just ran the process
and returned, unconditionally treating any outcome as success. That means the "job's best-effort
teardown might silently fail" risk everyone assumed was theoretical was, in the literal current code,
guaranteed never to be detected even when it happened — a failed `destroy` looked identical to a
successful one from the job's point of view, every single time.

**The fix, deliberately, is verify-before-delete, not persist-before-verify** — the exact discipline
this whole audit has been enforcing everywhere else, applied here for the first time to the single
highest-blast-radius operation in the panel. `removeAccount()` no longer deletes anything. It flips the
account to a new `Deleting` status via an atomic conditional `UPDATE ... WHERE status != 'deleting'`
(so two concurrent delete clicks can't both pass and both dispatch teardown), records
`deletion_started_at`, and dispatches `DeprovisionAccountJob` with the `Account` model itself — matching
this codebase's own established convention for every other "Run"-shaped async job (`MariadbUpgradeRun`,
`PanelUpdateRun`, etc. all pass the model, not scalars; the old scalar-args shape here was a documented
consequence of the delete-first ordering, not an independent design choice). The job now hard-deletes
the account **only** after `deprovisionRaw()` (fixed to actually check `$result->successful()`) confirms
real success; on failure it marks `DeletionFailed` with the script's error message and leaves the row
fully intact — visible in the WHM accounts list, with a **Retry Deletion** action that re-dispatches the
same job. A `Deleting`/`DeletionFailed` account's owner is blocked from logging in, the identical gate a
`Suspended` account already had (`FortifyServiceProvider`'s `authenticateUsing()` closure, one `whereIn`
widened). A `Deleting` account stuck past `DeprovisionAccountJob`'s own 90s timeout gets auto-flagged
`DeletionFailed` by `ResumeStuckSystemRuns`'s existing sweep — added as its own dedicated method rather
than forced into the shared `CONFIGS` array shape (which assumes a generic `started_at`/`completed_at`
pair that doesn't fit `Account`'s own semantics), following the same "safe to blind-fail" reasoning that
already governs everything else in that sweep except the one documented exception
(`AddonDomainConversion`, whose underlying script genuinely cannot be re-run — `DeprovisionAccountJob`'s
teardown steps, by contrast, are all safe to retry, so this fits the common case, not the exception).

One test failure surfaced on the first full-suite run after this change —
`ResellerScopingTest::test_reseller_can_destroy_own_account` — asserting the old immediate-hard-delete
behavior; updated alongside the four other test files (`DeprovisionAccountJobTest`,
`AccountControllerTest`, `AccountCreationTest`, `AccountApiTest`) that needed the same "row now survives
as Deleting under `Bus::fake()`" adjustment, plus new coverage in `AccountProvisioningServiceTest`,
`ResumeStuckSystemRunsTest`, and `AccountModifySuspendTest` for the new status transitions, the retry
action, the double-deletion guard, the stuck-sweep, and the login block. Full suite now at 2,440 tests
(up from 2,429), `pint --test` clean, deployed and live-verified via SSH on `panel-dev.knj.network`.

## Round twenty-two — a session-limit interruption, and the smallest finding count yet

Launched the same 8-category sweep the morning after closing the account-deletion gap. The session hit
Anthropic's own usage limit partway through — 7 of the 8 category agents (and their internal sub-agents)
failed mid-run with a session-limit error, arriving as a wave of near-simultaneous failure notifications.
No code was left in a broken state (the sweep agents are read-only), so the only casualty was incomplete
coverage: only the check-then-act lock-race category had finished before the limit hit, with one real,
unfixed finding (`SslController::retry()`, in both the `Account` and `Whm` namespaces, missing the same
lock `PhpVersionController::retry()` got in round twenty-one). Everything else paused until the limit
reset a few hours later.

On resume, the SSL retry fix went in first — same `Cache::lock("ssl-retry:{$site->id}", ...)` pattern as
`PhpVersionController::retry()`, guarding against two near-simultaneous "Retry" clicks dispatching two
overlapping certbot/ACME runs for the same site. One subtlety surfaced writing the regression test:
`abort_if($condition, 404)` throws `NotFoundHttpException`, which — because Symfony's `HttpException`
extends `\RuntimeException` — was getting silently swallowed by the account-side controller's own
pre-existing `catch (RuntimeException $e)` (kept for an older regression test covering a failed job
dispatch), turning a correct 404 into a misleading error redirect. Fixed by catching
`HttpExceptionInterface` first and rethrowing, ahead of the broader `RuntimeException` catch.

The remaining seven categories were then relaunched. Five came back clean — persist-before-verify across
mail/DNS/DB services, security/files/software services, and controllers directly, plus DB-constraint
create-race completeness (Laravel 13's `createOrFirst()` was confirmed, by reading the vendor source
directly, to self-heal a losing race via savepoint-wrapped retry even where no app-level `catch` exists)
— and IDOR/authorization came back clean for a fifth consecutive round, the strongest run yet for the
category where a real bug would matter most. One of the seven relaunched agents (persist-before-verify
security/files/software) hit round twenty-one's known "returns an interim non-answer instead of a final
result" failure mode — it had fanned out its own sub-agents and stopped mid-wait rather than
synthesizing; a follow-up message telling it to stop delegating and answer directly got a real final
report on the second pass.

Two more real findings came out of the two categories that weren't clean:

**`NginxSettingsService`'s single shared-lock design had a read-outside-the-lock gap.** `apply()` locks
only the write; three callers (`WafService::applyGlobalSettings()`, `InstallWafEngineJob`,
`PanelCertService::issue()`) that rebuild the shared nginx snippet from *whatever's currently in
Settings* — rather than a specific new value a form just submitted — called `currentSettings()` before
ever acquiring that lock. A concurrent `NginxSettingsController::update()` landing in between could
acquire the lock first, write its own new values, and persist its own Settings row; the other caller
would then acquire the lock second and write the file back from its stale pre-update snapshot, silently
reverting the winner's just-applied setting in the live nginx snippet even though Settings (and the WHM
page) still showed the new value as current. New `applyCurrentSettings()` reads `currentSettings()`
*inside* the same lock as the write, closing the gap; all three callers switched over.

**`AppInstallation` could get stuck at `Installing` forever with zero admin visibility.** This one was
already known and already deliberately excluded from `ResumeStuckSystemRuns`'s blind-flip sweep — same
reasoning as `AddonDomainConversion`, since `AppInstallerService::runInstall()` only knows how to roll
back what it created when it fails through its own normal code path, and a blind status flip on a
hard-killed row could leave a real database/database-user/files-on-disk orphaned and invisible. But
unlike `AddonDomainConversion`, which gets its own urgent, non-mutating admin alert, `AppInstallation`
got nothing at all — the class's own doc comment said "tracked separately," and there was no separate
tracking anywhere. Added `alertStuckAppInstallations()`, mirroring `alertStuckAddonDomainConversions()`
exactly: no schema change needed, since `beginInstall()` sets `status = Installing` synchronously at row
creation, so `created_at` already means what a dedicated `started_at` column would.

```
round:    initial  6   7   8   9  10  11  12  13  14  15  16  17  18  19  20  21  22
findings:   many   4  ~30  9  13  13   5   2   2   0   2  ~20  ~15  15  ~45  ~24  ~65   3
```

Three findings is the smallest count since round fourteen's zero — not because the codebase is
necessarily any cleaner than round twenty-one left it (three genuinely different, narrow gaps, not a
systemic pattern this time), but it's the first round in a while where the *majority* of categories
came back clean on a fresh, from-scratch read rather than finding new propagation gaps. Full suite now
at 2,446 tests (up from 2,440), `pint --test` clean, deployed and live-verified via SSH on
`panel-dev.knj.network`.
