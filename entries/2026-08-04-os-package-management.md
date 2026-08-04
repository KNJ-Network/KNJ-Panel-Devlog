# Phase 53 - OS Package Management, and Software & PHP Closed Out

One item was left in Software & PHP — an OS package management UI. The System Updates page only
applies updates to packages already installed, so installing something new was a real gap. This
adds searching the server's package index from the panel and installing on demand, queued with the
same live terminal-style log System Updates uses.

The interesting design question was how far to let an admin-chosen package name reach. The answer
follows the same reasoning as the Database Browser's query runner: this is an admin-only,
high-trust action, so there's no reason to fence it down to an allowlist of "safe" packages. The
privileged side still independently validates the package name and confirms the package exists in
the configured repositories before anything installs, rather than trusting the front end.

One deliberate performance call: search does not refresh the package index first, because that
would add seconds to every search for no benefit, since the index is already kept current by System
Updates and routine automatic updates. Installed-status per result is a fast lookup against the
installed list rather than a nested loop.

Live-verified on a development server with a harmless, side-effect-free package: searched, showed
as not installed, installed for real with the live log showing the package manager's output, and
searching again correctly showed it installed, proving the status is read live. It was removed
afterward.

Software & PHP is now fully Live.
