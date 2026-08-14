# Phase 81 - Ground Truth Before a Single Line of Code

Upgrading the database server's major version is the kind of admin action where a wrong guess
doesn't just break a feature. So this one started with research: what does the official repository
definition actually look like today, what is the safe shutdown-and-upgrade sequence, and is any
existing backup mechanism in the panel good enough to lean on.

## The dry run that changed the design

The research pass's first guess at the repository setup turned out to be an outdated pattern.
Rather than trust it, the official setup script was run in its built-in dry-run mode, which prints
what it would write without changing anything. The real answer differed in ways that would have
quietly broken the feature. The signing-key verification sequence was read from the script's own
source. Every value is now fixed in the provisioning script, and nothing is fetched and executed
from a remote script in production.

## A real rollback path, not just a warning label

The existing backup engine's per-account database dumps aren't enough on their own to recover from
a failed major-version upgrade. So this feature adds a mandatory pre-upgrade snapshot: a
crash-consistent copy of the data directory plus a full logical dump as a second independent
recovery path. The snapshot can be restored from the same page, with the same explicit
confirmation warning other hard-to-reverse actions use.

Snapshot, upgrade, and restore stream their output live to the page, and run under the queue worker
rather than the web server's restricted process.

## What shipped

Only supported long-term-support releases are offered as targets, never a development build, never
a version already installed or older. The page was deployed and tested on the development server,
but a real upgrade was deliberately not run there, since other work depends on that database.

## The disposable box did its job

Instead, a release was cut and installed on a disposable test box. The first real run found a bug:
the panel's own database often lives in the same database instance the snapshot briefly stops, and
a live-log write in that window threw an exception that stopped the job, which is deliberately
never retried automatically. Nothing was damaged and no packages were left half-installed. The log
write is now best-effort, so a brief outage just makes the log look momentarily stale.

A second release with the fix was installed through the real self-update mechanism, and the whole
run was repeated: a real snapshot, a real major-version upgrade, and a real restore from the
snapshot, all clean. Restoring a snapshot rolls back the whole panel database to that moment, which
is what the confirmation warning already says, and good to see confirmed for real.
