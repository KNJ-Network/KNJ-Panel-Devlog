# Phase 69 - The App Installer Becomes a Real Catalog, and phpBB Fights Back

The App Installer has done exactly one thing since it existed: install WordPress, synchronously,
blocking the request for up to two minutes while wp-cli ran. Both roadmaps already flagged the gap
— a full catalog was always "next," not "done." This phase closes it: Drupal, Nextcloud, MediaWiki,
and phpBB join WordPress, each through its own real, official, non-interactive CLI installer — the
same "the real thing, not a reimplementation" standard already applied to phpMyAdmin and wp-cli —
and the whole install flow moves off the request thread onto the same queued-job-with-live-log
pattern Package Manager already established.

## The shape of it

A small `AppInstallerDriver` interface (`key`, `label`, `icon`, `latestVersion`, `install`,
`remove`, `supportsUpdates`) replaced what used to be `AppInstallerService`'s WordPress-only
hardcoded logic. `WordpressInstallerDriver` is just that same logic moved behind the interface,
regression-tested to behave identically — including its update-check and staging-clone features,
which stay WordPress-only on purpose, since none of the other four apps has an equivalent
single-command "export DB, rewrite URLs" tool. `AppInstallerRegistry` maps key to driver, resolved
through the container. `AppInstallation` gained a `log` column and an `Installing` status that's
now actually visible — a controller creates the row synchronously (so the "already installed at
this path" check stays race-free) and dispatches `InstallAppJob`, which streams the provisioning
script's own stdout into that column chunk by chunk, the browser polling while status is
`Installing`, exactly like the Package Manager page already does.

Drupal, Nextcloud, and MediaWiki went in cleanly, each with its own small real bug caught by
actually running the install rather than trusting the code: Drupal's generated database password
had symbols in it, and drush's `--db-url=mysql://user:pass@host/db` doesn't reliably URL-decode a
password containing `@` or `:` — fixed globally by generating DB passwords alphanumeric-only rather
than trying to get the encoding right on both sides. Nextcloud's real 280MB archive has ~30,000
files and legitimately ships internal symlinks (`.js.map.license` files deduped against siblings) —
the archive-safety validator added in Audit #07 blanket-rejected all symlinks, which was too broad;
fixed to validate the symlink's *target* instead (reject absolute paths and `..` traversal, allow
safe in-tree links), and the install's own process timeout needed raising once actual timing showed
no real headroom at the old value. MediaWiki's release server flatly 403s any request with no
identifying User-Agent header — an easy, boring fix once the response body said so.

## phpBB, or: three ways to be wrong about a real installer

phpBB looked like the same shape of work — download, extract, run the CLI installer with a config
file — and turned into the most stubborn part of this phase. Three separate failures, each only
visible from a real install against a real server, each traced back to source rather than guessed:

**The version was wrong before anything else ran.** The plan's own research had confirmed a real
`.tar.bz2` URL pattern on SourceForge and moved on. What it hadn't checked: SourceForge's phpBB
project is frozen at 3.3.0. Every patch release since — up through 3.3.17 as of this writing — only
ever shipped as a GitHub git tag, never re-uploaded there. phpBB's own current download host,
`download.phpbb.com`, sits behind a Cloudflare JS challenge that blocks plain `curl` outright, so
that wasn't a fallback either. The real fix: resolve the latest `release-X.Y.Z` tag from GitHub's
tags API, then build from it the way phpBB's own contributor docs describe — copy the `phpBB/`
subdirectory out of the tag archive, then `composer install` to populate `vendor/`, the identical
official-build step Drupal's driver already uses via `composer create-project`.

**The wrong CLI entrypoint.** `bin/phpbbcli.php install <config>` — the binary named in the original
plan, confirmed against real phpBB source — bootstraps phpBB's *regular application* container,
which unconditionally tries to resolve `%core.table_prefix%` from `config.php` to look up installed
extensions. On a fresh install, `config.php` doesn't exist yet, and this dies with
`"core.table_prefix" of type NULL` before the installer ever runs a single step. phpBB ships a
second, entirely separate entrypoint for exactly this situation: `install/phpbbcli.php`, its own
dedicated installer container (environment `installer`, no extensions), which is what a fresh
install actually needs.

**The config file's own shape was wrong.** phpBB's install command reads YAML and hands it straight
to Symfony's `Processor::processConfiguration()` — an API that expects a *list* of config sets to
merge, not one config set. A flat top-level mapping (`admin: {...}`, `board: {...}`, ...) gets its
own keys misread as separate list entries, so `admin.email` (a string) landed where the schema
expected the whole `email:` block (an array), and the installer failed with `"Invalid type for path
'installer.email'. Expected array, but got string"` — a genuinely confusing error until traced back
to how `processConfiguration()` actually consumes its input. The fix is one leading `-`: wrap the
whole config as a single-item YAML list.

Only after all three were fixed did phpBB's install actually reach its database step — and hit a
fourth, unrelated wall along the way during manual verification: 3.3.0 predates PHP 8.1's default
mysqli error-reporting change and never calls `mysqli_report(MYSQLI_REPORT_OFF)`, so any
expected-to-fail-silently "table doesn't exist yet" query during install throws an uncaught
exception on this server's PHP 8.5. Building from the current 3.3.17 tag (already necessary for the
version bug above) fixes this too — confirmed by reading the driver source directly rather than
assuming.

Every one of these four fixes shipped only after a genuine install against `panel.dev.knj.network`
actually failed with them, was diagnosed from the real error and the real upstream source — never
guessed — and then re-verified with another real install before moving on. The final run: 13/13
phpBB installer steps, the forum served correctly at its real URL, cleaned up with zero files or
databases left behind.

## What's next

The catalog is done — five apps, each real, each async with a live log. Nothing else in the
Software section remains partial.
