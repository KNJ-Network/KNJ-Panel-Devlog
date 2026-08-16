# Phase 86 - Restoring the One File, Not the Whole Account

Backup Restoration has always meant one thing: wipe the account's home directory and re-extract
the whole `files.tar.gz`. Correct, but blunt — the roadmap's own item for per-file/per-directory
restoration sat unbuilt for a while precisely because it needed a different mechanism, not just a
smaller version of the existing one.

## Browsing an archive without a subprocess for every click

A backup's `files.tar.gz` roots at the account's system username — `jsmith/public_html/index.php`,
and so on for every domain and addon domain under that account's home. Listing what's inside it
doesn't need `sudo` at all: `backup-account`'s own privileged script already leaves the whole
backup directory group-readable by this app's own php-fpm user once a backup completes, so a plain
`tar tzf` is enough. Each level of the browse view is built by inferring directories from deeper
archive members rather than trusting the archive's own directory entries — robust regardless of
whether tar wrote an explicit entry for a given folder or not.

## Additive, not wipe-and-replace

The existing full restore deletes `/home/$USERNAME` outright before re-extracting — correct for
"put this account back exactly as it was," wrong for "I fat-fingered one config file." The new
`restore-account-path` provisioning action extracts only the named member (or, for a folder,
everything under it) straight into place, `chown`/`chmod`'d back to the account's own user,
nothing else on disk touched.

The one new input surface this adds — a path string coming straight from the web request, unlike
every other backup action, which only ever operates on an admin- or app-chosen whole backup — got
its own validation on both sides: a charset-restricted regex plus an explicit `..`-segment reject
in PHP, then the exact same two checks independently re-validated in the privileged script itself,
matching how every other user-influenced argument into that script has been handled since audit
#07.

## What shipped

A "Browse" link next to every completed account backup, both admin and account-owner sides —
breadcrumb navigation through the account's own file tree, a "Restore file" or "Restore folder"
button on every entry, each with its own confirm modal spelling out exactly what does and doesn't
get touched.

## Next

Continuing down the gap list.
