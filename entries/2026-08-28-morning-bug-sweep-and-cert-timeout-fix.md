# Phase 135 - A Morning of Bug Reports, and One That Took Two Passes

A batch of real usage reports from a production-style install, all fixed in one sitting rather than
piecemeal — nine separate issues, spanning account creation, dashboards, safety guardrails, and
update UX.

## The batch

**Create Account could no longer attach a hosting account to an existing login.** Every account had
always implied a brand-new login, which doesn't hold once one person legitimately owns more than one
hosting account. The account-creation form now offers a straight choice — create a new login, or
attach this account to one that already exists — and the provisioning layer generates a fresh system
username either way, since one login can now own several accounts, each still needing its own
distinct system identity.

**The dashboard and stats bar only ever showed one disk.** A server with `/home` on a separate
mount from `/` was silently only reporting `/`'s usage, everywhere in the UI. Both now detect a
separate `/home` mount (by comparing the backing device in `/proc/mounts`) and, when present, show
two bars instead of one — the fix a genuinely separate-disk server actually needed, not just wider
bars on the existing single graph.

**The Database Browser let anyone run arbitrary queries against the panel's own system databases.**
Listing them alongside customer databases is fine and useful; letting a query be run against them
was a live foot-gun — a single destructive statement there could take down the panel's own tables,
not just a customer's. System databases are still listed for visibility; "Run Query" is now replaced
with a plain "System" label for those rows.

**Database Maintenance's process-kill button could kill the panel's own database connection.** The
Active Processes list shows every live MySQL connection, including the panel's own. Killing an
arbitrary row is a legitimate admin tool for customer connections; killing the panel's own
connection is just self-inflicted downtime. Kill now checks the target process's owning DB user
against the panel's own configured user and refuses (with a "Protected" label instead of a live
button) rather than letting an admin discover this the hard way.

**System Updates buried its own progress indicator below the fold.** The "last run" terminal output
was rendered *after* the full pending-updates list, so on a server with a long update queue it looked
like nothing was happening while a real update was actually running just off-screen. Moved above the
list, added a second "Update Now" button at the top next to "Check Now" (so starting an update
doesn't require scrolling past the whole table first), and added a per-package "needs reboot" column
plus a reboot banner/button sourced from `/var/run/reboot-required` and its `.pkgs` companion — real
OS-level state, not a guess.

**Security Scan's Recommended Actions pills had a lot of dead space under the text.** One `flex`
alignment fix (`items-start` instead of the implicit stretch) — small, but it had been visibly wrong
on every load.

## The one that needed a second pass: Service Certificates reporting no hostname on a server that clearly had one

Shipped the batch above as v0.16.81, including a fix for Service Certificates showing "No hostname
set yet" on installs where a real hostname and certificate were already live — a reconciliation sweep
(`SyncPanelCertSettings`, run every fifteen minutes) that checks for a certificate already on disk and
backfills the setting the panel's own issuance flow would have written, for exactly the case where a
domain was known at install time and `bootstrap-server.sh`'s own certbot call issued the real cert
directly, bypassing the panel's own bookkeeping entirely.

Live-tested on a real production-style install, it still didn't work — and clicking "Set hostname" on
Server Setup, on a domain that was *already* set, silently did nothing either. Both symptoms pointed
the same direction once the underlying provisioning action was read start to finish: `issue-panel-cert`
was unconditionally re-running the *entire* mail-server and DNS-server installers — the same two
scripts `bootstrap-server.sh` itself runs once at install time — on every single invocation, including
the routine "certbot says this cert isn't due for renewal yet" no-op path that fires on every
reconciliation sweep and every repeat click. Individually each step of those two scripts checks
whether it's already done before doing it again, but running the whole thing end to end, apt calls and
service restarts included, still took over a minute even when nothing actually changed — comfortably
past the 3-minute process timeout and 3-minute-20-second job timeout the certificate-issuance path had
been given, sized back when this action was assumed to be a quick certbot call and a vhost rewrite,
not a full infrastructure re-provision.

Fixed with a plain marker file (`/etc/knjpanel/panel-mail-dns-domain`, written once the mail+DNS setup
has actually run for a given domain) gating that whole block — the first call for a new domain still
does the real work and gets real headroom to do it in (process timeout raised from 180s to 1700s, job
timeout from 200s to 1800s, comfortably under the queue worker's own hourly recycle point); every call
after that for the same domain finishes in a few seconds, the way the no-op case always should have.
Verified directly against a production-shaped install: cold (no marker yet) took just over a minute
end to end; the very next call, marker present, took four seconds.

Shipped as v0.16.82.
