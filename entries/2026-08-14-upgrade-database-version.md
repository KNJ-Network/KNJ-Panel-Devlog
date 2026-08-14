# Phase 81 - Ground Truth Before a Single Line of Code

Upgrading MariaDB's own major version is the kind of admin action where a wrong guess doesn't just
break a feature — it can take the whole server's data with it. So before writing anything, this one
started with pure research: what does a real, current MariaDB Foundation repository definition
actually look like today, what's the safe shutdown-and-upgrade sequence, and is there any existing
backup mechanism in this panel already good enough to lean on.

## The dry run that changed the design

The research pass's first guess at the repository URL turned out to be a plausible-sounding but
outdated pattern. Rather than trust it, the actual official repository setup script was run directly
against the dev server in its built-in dry-run mode — `--write-to-stdout`, which prints exactly what
it would write without touching the filesystem or importing anything. The real, current answer was
different in a way that would have quietly broken the feature if guessed: a different URL host
entirely, plus a package-priority pin file the initial research hadn't surfaced at all. Both LTS
targets this panel offers were checked this way, and the exact signing-key fetch-and-verify sequence
was read straight out of the script's own source rather than assumed. Every one of those values is
now a fixed literal in the provisioning script — nothing is piped from a remote script live in
production.

## A real rollback path, not just a warning label

The existing backup engine's own database dump — one `mysqldump` per account database, no system
schema, no users or grants — isn't enough on its own to recover from a major-version upgrade gone
wrong. So this feature adds its own: a mandatory pre-upgrade snapshot that stops the database
briefly, takes a crash-consistent copy of the whole data directory, and pairs it with a full logical
dump as a second, independent recovery path. That snapshot can be restored from the same page later,
with the same explicit confirm-modal warning every hard-to-reverse action in this panel uses — never
a native browser popup.

Snapshot, upgrade, and restore all stream their output live to the page as they run, following the
same pattern already established for package installs and system updates — except this one runs
under the queue worker's own unrestricted process, not the web server's sandboxed one, since writing
a new apt repository file lands outside what that sandbox is allowed to touch.

## What shipped

Only real, currently-supported LTS releases are ever offered as a target — never a rolling or
release-candidate build, never a version already installed or older. Deployed and tested against the
dev server's own render of the page — but the one thing deliberately not risked on it was triggering
a real upgrade against a database every other feature this session depends on staying healthy.

## The disposable box did exactly its job

Rather than test the real thing against the dev server, a fresh release was cut and installed on this
project's own disposable test box instead — a real, standalone install with nothing to lose. First
real run found a genuine bug: this panel's own database usually lives in the very MariaDB instance a
snapshot or upgrade briefly stops. A live-log write landing in that exact window threw an uncaught
exception that took the whole queue worker down with it — and because the job was deliberately
configured to never auto-retry (retrying a half-finished server upgrade automatically is its own kind
of dangerous), Laravel just gave up on it, leaving the run stuck at "Running" forever with no real
work ever done.

No actual damage — dpkg was clean, MariaDB was untouched, nothing was left half-installed. The fix
was small: that log write is now best-effort, so a connection hiccup during the brief outage just
means the log looks momentarily stale, never a dead job. Cut a second release with the fix, upgraded
the test box again through its own real self-update mechanism, and ran the whole thing again from
scratch: a real snapshot, a real `10.11.14 → 11.4.12` upgrade, and a real restore back from that
snapshot — all three genuine, all three clean, the queue worker never crashed once on the second
pass. Restoring a snapshot turned out to roll back the whole panel database to that exact moment, not
just the schema — expected once you think about where the data actually lives, and already exactly
what the confirm-modal's own warning says, just satisfying to see confirmed for real rather than
assumed.

## Next

Continuing down the gap list.
