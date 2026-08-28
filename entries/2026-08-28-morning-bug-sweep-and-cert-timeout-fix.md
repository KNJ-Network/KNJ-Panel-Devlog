# Phase 135 - A Morning of Bug Reports, and One That Took Two Passes

A batch of real usage reports from a test install, all fixed in one sitting rather than piecemeal:
nine separate issues spanning account creation, dashboards, safety guardrails and update UX.

## The batch

**Create Account could no longer attach a hosting account to an existing login.** Every account had
implied a brand-new login, which does not hold once one person owns more than one hosting account. The
form now offers a straight choice, and the provisioning layer generates a fresh system username either
way, since each account still needs its own distinct system identity.

**The dashboard and stats bar only showed one disk.** A server with its home directory on a separate
mount only reported the root filesystem. Both now detect a separate home mount and show two bars.

**The Database Browser allowed queries against the panel's own system databases.** Listing them is
useful, but running a query there was a foot-gun for the panel's own tables. They are still listed,
and "Run Query" is replaced with a "System" label for those rows.

**Database Maintenance's process-kill button could kill the panel's own connection.** Kill now refuses
for the panel's own database user and shows a "Protected" label instead.

**System Updates buried its progress indicator below the fold.** The "last run" output is now above
the pending list, a second "Update Now" button sits next to "Check Now", and a per-package "needs
reboot" column and reboot banner use the operating system's own reboot-required state.

**Security Scan's Recommended Actions pills had dead space under the text.** One CSS alignment fix.

## The one that needed a second pass: Service Certificates reporting no hostname

The batch shipped as v0.16.81, including a fix for Service Certificates showing "No hostname set yet"
on installs that already had a hostname and certificate: a periodic reconciliation backfills the
setting where the install script had issued the certificate directly, bypassing the panel's own
bookkeeping.

Live testing on a test install showed it still did not work, and clicking "Set hostname" on a domain
that was already set silently did nothing. Reading the provisioning action end to end showed it
re-ran the entire mail-server and DNS-server setup on every invocation, including the routine
"certificate not due for renewal" path. Each step is idempotent, but the whole run took over a
minute, past the process and job timeouts that had been sized when this was a quick certificate call.

It was fixed with a marker, written once the mail and DNS setup has run for a given domain, that gates
that block. The first call for a new domain still does the real work with proper timeouts; later calls
finish in seconds. Verified: the cold call took just over a minute, the next call four seconds.

Shipped as v0.16.82.
