# Phase 160 - Three Things the Real Servers Wanted

A round of feedback from the actual 4-server production stack — one live-reproduced bug, one
explicit product-scope decision, and one feature request from the day's own manual update rollout.
All three shipped together.

## phpMyAdmin: installed everywhere except where it mattered

phpMyAdmin has worked on `panel-dev` since it shipped weeks ago. On the real Main server it
returned a plain 404. The nginx locations were fine — `issue-panel-cert` rewrites the whole system
vhost unconditionally on every hostname/cert re-confirm, so those self-corrected the moment this
server's copy of the script caught up with the code that added them. What never happened was the
actual package install: `bootstrap-server.sh` only installs phpMyAdmin once, during a server's own
first provisioning run, and this server had been bootstrapped before that line existed. Nothing
anywhere ever revisits that gap once a server is already up — confirmed directly on the box,
`/usr/share/phpmyadmin` genuinely wasn't there.

The fix reuses a pattern this same action already leans on for something unrelated: Roundcube's
`des_key` and directory-permission self-heal, both unconditional checks that run on every single
call to `issue-panel-cert`, cheap enough not to care that they usually find nothing to fix. Added a
matching one for phpMyAdmin — `dpkg -s phpmyadmin`, and if it's missing (main role only), run
`setup-phpmyadmin.sh`, which was already fully idempotent and safe to call outside its original
one-time bootstrap context. Any already-provisioned Main server picks this up automatically the
next time an admin re-confirms its hostname — no SSH, no manual package install, no re-running the
dangerous parts of bootstrap.

## No more surprise databases

While testing Deploy from Git on the real `knj.network` account, a MySQL database named `knj`
turned up that nobody had created. Traced it to `AccountProvisioningService::provision()` quietly
calling `AccountDatabase::create()`/`DatabaseUser::create()` on every single account, matching real
cPanel/WHM's own default. The user's call was direct: an account owner should choose to add a
database, not find one already sitting there.

Removed the auto-create from both places it happened — `provision()` (WHM Create Account) and
`convertAddonDomain()` (Convert Addon Domain to Account) — including the underlying `CREATE
DATABASE`/`CREATE USER` calls in the provisioning script itself, not just the panel-side rows, so
there's no orphaned real database left behind that nothing tracks. Transfer Tool and both cPanel
import paths were checked and deliberately left alone: those *import* an existing database, which
is a different thing entirely from inventing a blank one nobody asked for.

## Update all the things, from one page

The DNS zone glue fix two phases ago had to be rolled out to all four production servers by hand —
log into each one's own Panel Updates page, click Update Now, wait, repeat. Tedious enough to be
worth fixing properly rather than just enduring again next time.

Two pieces, both riding the same two-key mutual-auth channel every other cross-server call in this
codebase already uses:

**A real heartbeat.** DNS-only satellites already had one — `PushZoneMembershipJob`'s zone push
doubles as proof the linked app itself is alive, not just reachable on port 22. Mail-only never had
anything like it. Built a small, role-agnostic version (`ServerPingController`, works for either
role) and wired it into `ServerHealthCheckService` right after the existing TCP probe succeeds —
best-effort, swallows every failure, since a stale version number is a much smaller problem than
turning a working reachability check into a flaky one. It reports back `panel_version`, stored on
the `Server` row, shown on Manage Servers with a "Latest" pill once it matches the manifest.

**Actually triggering the update, remotely.** A satellite's self-update restarts its own PHP
process mid-request, so this could never be a synchronous call-and-wait the way most cross-server
actions in this codebase are. `applyPanelUpdateAll()` creates one tracking `PanelUpdateRun` per
linked server (`server_id` set — null still means "this box's own run" everywhere that mattered
before this column existed) and dispatches `DispatchRemotePanelUpdateJob`, which tells the satellite
to start its own ordinary `applyPanelUpdate()` flow, completely unchanged, and returns immediately.
`PollRemotePanelUpdateJob` then re-dispatches itself every few seconds, mirroring the satellite's
real progress back into Main's tracking row, until it's done — capped deliberately just under
`ResumeStuckSystemRuns`' own 1950-second stuck-run threshold for `PanelUpdateRun`, so a genuinely
wedged satellite gets a clear "gave up waiting" from this job first, not a generic blind-flip from a
sweep that has no idea it's looking at a remote-tracking row.

The button only appears when there's actually a linked server to update — nothing changes on a
single-server install.

Tested (2532/2532).
