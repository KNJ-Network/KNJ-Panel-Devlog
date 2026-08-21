# Phase 107 - A Webmail Login Bug With Two Separate Causes, and a Session That Leaked Sideways

Two unrelated bugs, found live and fixed in the same session — one in the mail stack, one in the
admin "log in as user" feature. Both are the kind that only show up once real traffic, or a second
browser tab, actually exercises the path.

## Webmail login broke, and the fix was two bugs deep

Reported symptom: freshly created mailboxes couldn't log into webmail on either Main or a linked
Mail Only satellite. The diagnosis that cracked it open was the user's own suggestion — go back to
Main, point mail at the local server instead of the linked satellite, and see if login works there.
It did. That narrowed the search from "mail routing is broken" to "something about how a mailbox's
password gets provisioned is broken," which turned out to be true in a way that had nothing to do
with Mail Only at all.

**Bug one, in the shared provisioning script every install uses**: `doveadm pw` doesn't behave like
a normal "read stdin, hash it" tool. Even with no terminal attached, it still expects input shaped
like its own interactive prompt — the password, then a second confirmation entry — and if it only
gets one value, it silently hashes something other than what was intended and exits `0` anyway,
with the actual complaint (`Passwords don't match!`) going to stderr where nothing was watching.
First fix attempt just added a trailing newline — read as correct, shipped, still broken on the
next live login test. Confirmed directly on the box: feeding `doveadm pw` a single value produces a
bogus hash with a clean exit code. Real fix: feed it the same plaintext twice.

**Bug two, specific to Mail Only satellites**: once the first bug was fixed and Main's own mailboxes
worked again, the satellite still didn't. `journalctl` on the satellite's Dovecot showed
`passwd-file uid=5000 gid=5000 /etc/dovecot/mail-users:open(...) failed: No such file or directory`
— Dovecot 2.3's `passwd-file` userdb driver doesn't parse `key=value` tokens out of its own `args`
string the way `passdb`'s `scheme=` prefix does; the whole string gets treated as one literal
filename. `setup-mail-server.sh` had been writing `args = uid=5000 gid=5000
/etc/dovecot/mail-users`, valid-looking but wrong. Fixed by dropping the redundant `uid=`/`gid=`
prefix — the passwd-file's own per-line fields already carry that.

Both fixes verified with real IMAP `LOGIN` commands succeeding on freshly created mailboxes, on both
`panel.dev.knj.network` and `mail.dev.knj.network` — the satellite's fix delivered through a real
cut release (`v0.16.29`) and its own "Update Now" button, the same path a customer would use.

One loose end, not blocking anything: a single disposable test mailbox never showed up in the
satellite's regenerated auth map across several separate trigger attempts, despite a DB row
identical in every column to three working mailboxes and no exception ever thrown. Deleted rather
than left in a broken state; flagged to revisit if it recurs on something real.

## "Log in as user" was quietly hijacking the admin's own session

Separately: an admin using the "log in as user" button, then switching to another already-open
Controller tab without first clicking "Return to admin," found that second tab 403'd —
*"This account cannot access the controller area."* Traced to the actual design: the controller
area and account area are one Laravel app on two ports, sharing exactly one session cookie and one
auth guard. Impersonating a hosting account owner was a plain login-as-them, which replaced the
admin's own identity for the *entire* session — every other tab, every other request, until the
admin explicitly ended it or logged out and back in.

Fixed by giving the account area its own second guard, authenticated only for the impersonated
owner. The admin's own session is never touched by starting an impersonation, so any other open
tab keeps working the whole time. Ending it logs out the second guard only — no full session
invalidation, which would have logged out every other tab too and defeated the point.

Live verification of that fix immediately surfaced a smaller one it introduced: the amber "you're
impersonating someone" banner, which reads its "should I show" state from the new guard but its
"whose name do I print" state from whichever guard is default on that particular request, ended up
appearing on the admin's own genuine Controller Dashboard in the second tab — correctly not
403'ing anymore, but now showing "Logged in as [the admin's own name] — actions here affect their
account" on a page where no impersonation was active at all. Scoped the banner to the account port
specifically, so it only ever renders on the tab that's actually mid-impersonation.

## Verified

Both fixes: full local suite and `pint --test` green throughout. Live in a real browser: impersonate
from one Controller tab, confirm a second already-open tab keeps working with zero re-auth and no
banner leak, click "Return to admin," confirm a clean redirect with the banner gone and the admin's
own session untouched throughout.
