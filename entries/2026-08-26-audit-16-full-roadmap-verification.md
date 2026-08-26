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

An eighth re-audit round against this fix set is in progress before any version bump or release cut.
