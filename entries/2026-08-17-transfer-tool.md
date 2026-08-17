# Phase 92 - Transfer Tool

The last genuinely-missing piece of the original migration-tools roadmap line: pulling one or more
accounts straight from a live, remote cPanel/WHM server over SSH, rather than requiring someone to
manually generate a `cpmove-*.tar.gz` and upload it through Restore a cPanel Account first. This is
what WHM itself calls its own "Transfer Tool."

## Reuse, not reinvention

This feature's own job ends the moment a valid archive is sitting on this server. Everything after
that — validating the archive, detecting the domain, listing the databases inside it, creating the
account, restoring files and databases — is the exact same `CpanelImportService` +
`CpanelImportRun` + `ImportCpanelAccountJob` pipeline Restore a cPanel Account already uses,
completely unchanged. The only new code in `CpanelImportService` is one small, additive refactor:
`stageArchive(UploadedFile $file)` now delegates its validation/staging tail to a new public
`stageFromLocalArchive(string $archivePath)`, so a locally-SSH-pulled file can feed the same path
an uploaded one does.

On the remote end, it runs the same real tool WHM's own Transfer Tool and Full Backup feature use —
`/scripts/pkgacct` — rather than reimplementing account packaging. Account enumeration reads
`/etc/trueuserdomains`, WHM's own maintained one-primary-domain-per-account file.

## Private key only, and `sudo` everywhere

No password authentication option — key-based auth is what cPanel's own Transfer Tool
documentation steers admins toward for exactly this job, and skipping passwords entirely avoids
both a new `sshpass` dependency and a password-in-a-temp-file handoff for a credential this
sensitive.

Every remote command — `pkgacct`, reading `/etc/trueuserdomains`, the post-`pkgacct` `chmod` so a
non-root SSH user can still download the result — runs through `sudo` on the remote end rather than
assuming the SSH login itself is root. This was a real design correction mid-build, not a starting
assumption: the first pass authenticated as `root` directly, which is exactly backwards from how a
security-conscious server should actually be set up. Root SSH login should stay disabled; a regular
account with passwordless sudo for the handful of commands this needs is the better-practice
default, and it costs nothing here since `sudo` behaves identically whether the connecting user
already is root or not.

## Two bugs only a real live test would ever catch

Both were found running an actual pull against a real remote box, and neither is reachable from the
existing unit/feature test suite — one needed real compiled Blade→HTML output, the other needed an
actual browser `<textarea>` submission.

**A Blade escape collision.** The sources table rendered `kritchie{{ $source->hostname }}:22` —
literal, unparsed Blade syntax — instead of `kritchie@62.31.247.86:22`. The three adjacent echoes
(`{{ $source->ssh_username }}@{{ $source->hostname }}:{{ $source->port }}`) happened to contain the
exact byte sequence `}}@{{` that Blade's compiler treats as its own literal-`{{ }}` escape
directive (meant for letting JS frameworks like Vue coexist in the same template). The compiler
can't distinguish "I meant three separate echoes" from "I meant to print a literal `{{ }}`" — it
just matches the pattern. Fixed by collapsing the three echoes into one interpolated string.

**SSH keys corrupted by browser textarea CRLF submission.** Clicking "Test" on a freshly-added
source failed with an opaque `error in libcrypto` from OpenSSH's key parser. A browser `<textarea>`
submits its content with CRLF line endings regardless of what was actually typed or pasted in —
OpenSSH's parser only accepts LF. Fixed by normalizing line endings at the point the key is written
to its short-lived temp file, not just at intake, so a key already stored corrupted from an earlier
submission self-heals the next time it's used rather than requiring the source to be re-added.

## Live-verified end to end

Built a synthetic-but-format-faithful `/scripts/pkgacct` on a real, independently-controlled box
(same approach used for every other "no real cPanel server available" gap this project has hit) —
producing a real `cpmove-*.tar.gz` shaped exactly like a genuine one, with a website file and a
database containing a known marker row. Ran a full pull through the real browser UI: connect, list
accounts, transfer, watch the pull complete, follow through to the linked import run completing,
then confirmed on the target server that the file and the database row both landed correctly,
byte-for-byte, with the new account's own system user owning them.

## Next

Convert Addon Domain to Account — the one remaining item on the original migration-tools roadmap
line, promoting an addon domain into its own top-level account.
