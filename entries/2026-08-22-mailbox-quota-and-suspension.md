# Phase 123 - Per-mailbox quota and suspension

The fourth item off the "genuinely zero" list. A per-mailbox quota value had existed since mail
first shipped, but only as a stored number: set once at creation, displayed read-only, never
enforced and never editable. Mailbox suspension did not exist at all.

## Two auth backends, two places to enforce both things

The panel's mail authentication runs one of two ways depending on topology. On Main, lookups query
the panel's own database directly. A linked Mail Only satellite has no such copy, so the auth-map
service pushes the full current mailbox set as a flat password file.

Both quota and suspension had to be wired into both paths:

- **Database path**: suspended mailboxes are excluded from the lookup, so login fails as "user
  unknown" rather than through a special suspended code path, and a quota rule is supplied only
  for capped mailboxes (0 stays genuinely unlimited).
- **Password-file path**: suspended mailboxes are filtered out before the file is built, with the
  same "no row, no login" outcome, and a quota rule is added only for capped mailboxes.

Suspended mailboxes are left out of both sources entirely, so there is exactly one thing to get
right, not two in agreement.

## Enabling the quota plugin

The mail server's quota support ships in its core package and only needs turning on. It was added
to the fully managed settings drop-in the config action already regenerates, enforcing at delivery
time and reporting quota to IMAP clients, using the mail format's own size tracking, with no
separate quota database or service.

## What's account-side

Both features are self-service on the Email Accounts page. Suspension reads oddly self-service at
first, but is framed as a defensive tool for a compromised mailbox: lock out login while
investigating, without losing a message, since mail keeps arriving.

## Safety

Negative quotas are rejected, ownership is checked as for every other per-mailbox action, and
neither feature adds a new privileged provisioning action.

## Verified

14 new tests; full suite 1,759 and the code-style check green.

## Live-verify addendum: three real bugs, all satellite-related

Testing against the Mail Only satellite surfaced bugs the unit tests could not catch:

**Bug 1:** the provisioning script's line validator rejected the new quota field. It now accepts a
quota rule in exactly the expected shape, not free text.

**Bug 2:** quota silently did not enforce even once accepted, because the shared file is read for
both password and user lookups and an unprefixed extra field is ambiguous between them. Using the
mail server's documented convention for user-only fields fixed it.

**Confirmed end to end:** a disposable mailbox with a 1 MB quota accepted a first message and
rejected the next with a standard "mailbox full" bounce, as a real client would see. Test data was
cleaned up afterward.
