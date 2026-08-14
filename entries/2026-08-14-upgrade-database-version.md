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

## What shipped, and what deliberately didn't run yet

Only real, currently-supported LTS releases are ever offered as a target — never a rolling or
release-candidate build, never a version already installed or older. The code is built, tested, and
deployed to the dev server. What hasn't happened yet is triggering a real upgrade against that
server's own live database — every other feature built this session depends on that database staying
healthy, so that one step waits for an explicit go-ahead rather than being assumed.

## Next

Continuing down the gap list.
