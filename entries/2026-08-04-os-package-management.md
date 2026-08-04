# Phase 53 - OS Package Management, and Software & PHP Closed Out

One item left in Software & PHP — an OS package management UI — and it turned out to not
already exist under a different name. The System Updates page (Phase from earlier this project)
only ever applies updates to packages already installed; installing something genuinely new was
still a real gap, so this session built it: search the server's apt package index from the panel,
then install on demand, queued with the same live terminal-style log System Updates already uses.

The interesting design question wasn't the happy path — it was how far to let an admin-authored
apt-get install argument reach. The answer follows the same doctrine as the Database Browser's
query runner from a couple sections back: this is an admin-only, root-equivalent action, matching
the trust tier of Server Setup or a System reboot, so there's no reason to fence it down to an
allowlist of "safe" packages the way a lower-privilege surface might need to. The provisioning
script still independently re-validates everything anyway — the package name against Debian's
real naming rules, then a live `apt-cache show` check that the package actually exists in the
configured repositories before anything installs — never trusting that PHP's own validation was
enough, same as every other privileged action in this script.

One deliberate performance call: the search action does *not* run `apt-get update` before
searching. It's a synchronous, real-time part of a web request — every keystroke-driven search
would otherwise cost several seconds refreshing the package index for no real benefit, since the
local index is already kept current by System Updates' own "Check now" and by unattended-upgrades'
regular apt timers. Installed-status per result is a hash-set lookup against `dpkg-query`'s output,
not a nested loop — this server easily has a couple thousand packages installed, and checking each
search result against all of them one at a time would make even a narrow search feel sluggish.

Live-verified with a genuinely disposable, side-effect-free package (`cowsay`) — searched, showed
up correctly marked not-installed, installed for real (confirmed via SSH: `cowsay` ran and drew its
cow), the run's live log showed the actual apt output including a `needrestart` service-deferral
notice, and searching again afterward correctly showed it as already installed — proving the
installed-flag is read live from `dpkg-query`, not cached. Removed it immediately after to leave no
residue, matching this project's standing rule for anything installed live purely to test a feature.

Software & PHP is now fully Live — the last remaining "Planned" row in that section is gone.
