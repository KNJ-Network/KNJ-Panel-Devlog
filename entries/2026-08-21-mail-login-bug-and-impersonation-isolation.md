# Phase 107 - A Webmail Login Bug With Two Separate Causes

Two unrelated bugs, found during live testing and fixed together: one in the mail stack, one in
the admin "log in as user" feature.

## Webmail login broke, and the fix was two bugs deep

Symptom: freshly created mailboxes could not log into webmail on Main or on a linked Mail Only
satellite. Pointing mail back at the local server and retrying showed login worked there, which
narrowed the search to how a mailbox's password is provisioned.

**Bug one, in shared provisioning tooling:** the mail server's password-hashing tool does not
behave like a plain "read stdin, hash it" command. It expects its input in the shape of its
interactive prompt, and with one value it silently produces the wrong hash while still exiting
successfully. The first fix attempt was not enough; confirmed directly on the test box, and
corrected by supplying the input in the shape the tool expects.

**Bug two, specific to Mail Only satellites:** once Main's mailboxes worked again, the satellite
still failed. The satellite setup had been writing a valid-looking but wrong argument string for
the mail server's user-database driver. Fixed by dropping the redundant options.

Both fixes were verified with real IMAP logins on freshly created mailboxes on the main test
server and the linked mail satellite, delivered through a real release and the satellite's own
"Update Now" button, the same path a customer would use.

## "Log in as user" was replacing the admin's own session

An admin using "log in as user", then switching to another already-open Controller tab without
clicking "Return to admin", found that tab refused access. Impersonation replaced the admin's
identity for the entire browser session instead of being scoped to the impersonated area.

Fixed so that impersonation is tracked separately from the admin's own sign-in: starting it never
touches the admin's session, so other open tabs keep working, and ending it only ends the
impersonation. Live verification surfaced one smaller issue the fix introduced: the "you're
impersonating someone" banner appeared on the admin's own dashboard in the second tab. The banner
is now scoped to the area that is actually being impersonated.

## Verified

Full local suite and code-style check green. Live in a real browser: impersonate from one tab,
confirm a second open tab keeps working with no banner leak, click "Return to admin," confirm a
clean redirect with the admin's own session untouched.
