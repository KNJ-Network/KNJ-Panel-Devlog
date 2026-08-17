# Phase 94 - Fix Everything, Then Audit It

Different instruction from every prior audit: go through everything first, fix whatever's found,
*then* run the audit — and it should come back clean, because anything it would have caught should
already be gone. Almost every past audit worked the other way around (review, find, fix, re-verify),
so this one meant genuinely front-loading the review instead of leaning on the audit itself to
surface problems.

## Going back over already-audited code, not just what's new

The instinct on a deadline is to review what changed since the last pass. That's not what got asked
for here — "check every line, the full thing," repeated more than once after the first pass came
back with real findings. So this one covered every controller, the whole services layer, all 182
views, all 62 models, and the entire ~6,000-line provisioning script — including plenty of code
that earlier audits had already looked at and passed.

That insistence paid off directly. A scoped review of the two newest privileged file operations
found the same bug in both: a root-run action writing into an account's home directory without
checking whether the destination was currently a symlink. An account owner has ordinary write
access to their own home over FTP or the File Manager — nothing stops them from replacing a
directory the next restore or install was about to write into with a symlink pointing anywhere on
the filesystem, and having root follow it right out of their own account boundary. Two instances,
easy enough to call it done and move on.

Instead, the whole script got swept for the same pattern on the theory that if it happened twice in
the newest code, it probably happened before too. It had — nine more instances turned up, several
in code multiple earlier audits had already passed as clean. All eleven got the same fix: a shared
`assert_path_within_home()` check, applied everywhere a privileged write follows a path an account
owner ultimately controls.

## The second-order bug in the "obvious" fix

Two long-lived secrets — a webhook token and a dynamic DNS update token — were sitting in the
database in plain text, despite each one being the *entire* authentication for its own public,
unauthenticated endpoint. The obvious fix is an `encrypted` cast. Applying it here would have
silently broken both features: an encrypted column only decrypts on read, and encryption uses a
random IV, so the exact `where('token', $incoming)` lookup each endpoint depends on would never
match ciphertext again. Worse, both tokens are shown persistently on their own settings page — the
account owner needs to be able to come back and re-copy the URL — so a hash-only design (fine for a
one-time API token) wasn't viable either.

The actual fix splits the two jobs onto two columns, the same shape Sanctum itself already uses for
`personal_access_tokens`: a deterministic hash column for the public endpoint's lookup, and the
original column kept for redisplay but now genuinely encrypted at rest. Verified with a real
round-trip against panel-dev's live database afterward, not just the test suite — create a row,
confirm the raw database value is ciphertext, confirm decrypting it back through the model matches
the original, confirm the hash-based lookup still finds it.

## What else turned up

A folder name typed into the File Manager could inject arbitrary nginx directives, because two
features (Directory Privacy, WebDAV) embed that name straight into a generated config block with no
restriction on what characters it could contain — completely reasonable for the File Manager itself,
since a folder called almost anything is fine for plain storage, but not once that name becomes
literal server config. Several database and mail passwords were briefly visible to any other local
user on the box via the process list for as long as the command handling them ran, closed by moving
every one onto a private input channel instead. A handful of leading-dash argument-injection gaps,
same class as an earlier audit's `dig` fix, turned up in three more places. And the self-server row
— the one every account creation depends on having exactly one of — could still be opened and saved
through the ordinary server-edit form, silently corrupting it with no attacker required, just an
admin clicking into a page the UI otherwise treats as safe.

Fourteen issues in total, every one fixed the same session, the full 1,414-test suite re-run clean
afterward, several of the higher-stakes fixes (the self-server guard, the folder-name charset check,
the token encryption) re-verified live against panel-dev itself rather than trusted to the test
suite alone. The audit proper, run against the fixed code, found nothing left.

## What's genuinely still open

A few things were deliberately left alone rather than guessed at: three of the five app installers
(Drupal, Nextcloud, phpBB) still pass their database password as a bare command-line argument the
same way WordPress and MediaWiki used to, but unlike those two, there's no confirmed-safe
stdin/env-var alternative for their specific CLI tools yet — that needs real research against each
tool before touching it, not a guess that could quietly break an installer. A handful of admin-only
config-write actions got flagged by one review angle as under-validated and cleared by another as
already tightly bounded — worth resolving with a dedicated trace before deciding either way. Both
carried forward rather than rushed.

## Next

The deferred installer-password items, and the wider-server-fleet re-verification this pass
deliberately left out of scope.
