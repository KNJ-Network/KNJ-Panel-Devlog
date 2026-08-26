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
