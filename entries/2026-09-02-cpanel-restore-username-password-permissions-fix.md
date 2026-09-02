# Phase 172 - What a Real Customer Restore Actually Breaks

Every phase testing the cPanel restore feature so far had used either a hand-built synthetic
archive or, later, one real ~975MB backup archive pulled specifically to verify the format. This
phase was the first time it ran against an actual customer's account, restored for real, with a
real website depending on the result. It broke in three different ways, none of which the earlier
testing had surfaced.

## Seven characters that mattered

The operator's own account was a real one — `visiontech336.com`, with an original 8-character
username, `visionte`. After the restore, the new account came out as `visiont` — one
character short. `AccountProvisioningService::generateUsername()` had always truncated to 7
characters, matching nothing in particular; the original username was 8. Nobody had noticed before
because nothing about a *synthetic* archive cares what the username matches — a fresh throwaway
domain gets a fresh throwaway username and nothing else references it. A real restore is different:
KNJ generates every database name and database username from the account's own system username
(`{username}_{suffix}`), so a 7-character username meant every single restored database came out
under a name one character removed from the one already baked into the site's own copied
`wp-config.php`. The fix itself was trivial — three `substr(..., 0, 7)` calls became `substr(...,
0, 8)` — but it explains a failure mode that looked, from the outside, like almost nothing about
the restore had worked.

## The password that was never going to match anyway

Fixing the username alone wasn't enough, and it was tempting to assume it would be. Every restored
database has always gotten a fresh, randomly generated password — reasonable on its face, matching
how the Database Wizard creates a brand new database. But a *restored* database isn't a brand new
one. Its whole point is that a config file somewhere on disk already has a password baked into it,
copied over verbatim as part of the same restore. A random new password was never going to match
that file no matter how well the username matched.

The fix follows a precedent already sitting in the same codebase: mailbox passwords aren't reset on
restore either. The archive's own `homedir/etc/<domain>/shadow` carries a real SHA-512-crypt hash per
mailbox, and `MailboxService::setPasswordHash()` transplants it directly rather than generating
something new — the original password just keeps working, because MySQL and Dovecot alike are happy
to accept an account created from a pre-computed hash instead of a plaintext. The natural question
was whether MySQL databases had an equivalent. They do: a real backup archive's `mysql.sql-auth.json`
carries a `mysql_native_password` hash per database username, hex-encoded for JSON safety (confirmed
by pulling the real archive and hex-decoding a sample by hand — this codebase has been burned before
by trusting a third-party description of an archive format instead of a real sample, so this got the
same treatment). MariaDB 10.11 — confirmed live on panel-dev — accepts `CREATE USER ... IDENTIFIED
WITH mysql_native_password AS '<hash>'` directly. Restoring a database now looks up that hash first;
when one's found, the original password is set with no plaintext ever touching KNJ Panel's own
database, and the site's already-copied config needs no editing at all. When no hash is found — a
malformed archive, an unsupported auth plugin — it falls back to the original behavior: a fresh
password, plus an automatic rewrite of any `wp-config.php` found referencing the old database name.

Transplanting a hash instead of generating a password means KNJ Panel genuinely doesn't know the
plaintext for that database user, which turned out to matter in two places that had always assumed
a `DatabaseUser`'s stored password was real and usable: phpMyAdmin's one-click sign-on, and allowing
a new remote-access host (both mint a fresh MySQL account using the stored password). A new
`password_known` flag on the model — defaulting true for every normal path — makes both show a
plain, honest explanation instead of quietly attempting a connection with a value that was never a
real password to begin with.

## The directory that could look at itself but not touch itself

The third bug was unrelated to credentials entirely. Deleting `wp-config-sample.php` — a completely
routine File Manager action — failed with a raw 500. The restored `public_html` directory had been
`chmod 750`'d, the same permission every other account home-directory-root path gets. Everywhere
else that's correct: the panel's own process only needs to *read* a home directory, never write into
it directly. `public_html` is the one exception — every file restored into it needs to stay writable
by the panel's own process afterward, through the group bit, exactly like every other provisioning
path already treats it. One inconsistent `chmod` call, fixed to match the rest. Alongside it,
`FileManagerService::delete()` picked up a `try`/`catch` around the actual `unlink()`/`rmdir()` call
— a plain PHP warning on a failed delete becomes a catchable exception under this app's own error
handler, and it had never been caught here, so any future permission mismatch will now surface as a
normal, readable error instead of a crash.

## What's still just a warning

The operator asked, reasonably, whether the restore should actively check for a username mismatch
and offer to fix it — then immediately answered their own question: matching the original 8-character
length should already prevent the mismatch from happening in the first place. That's almost certainly
right, so this phase added the cheap, honest version of the idea instead of a whole new confirmation
flow: a real backup archive's `cp/<username>` file name *is* the account's canonical original
username, so the restore now compares it against the newly generated one and logs a plain warning —
never a block — if they still don't match. Something to notice, not something to get in the way.

## Verified live, not just in the suite

2673/2673 passing (7 new tests: hash-found and hash-not-found database restore paths, the
username-mismatch warning firing and not firing, the phpMyAdmin guard, the remote-host guard, and a
real permission-denied directory reproducing the exact File Manager failure). Then live on
panel-dev: created a real account through the actual Create Account form against a domain whose
first eight characters would previously have collided with the old seven-character truncation,
confirmed the generated username came out as the full eight characters, confirmed the new account's
own dashboard loaded cleanly, then removed the test account through the normal UI flow. The real
`visiontech336.com` restore itself gets redone from scratch once this ships, this time against a
website with a database and a config file that were both restored to actually agree with each
other.

Tested (2673/2673).
