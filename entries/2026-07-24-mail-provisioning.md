# Phase 08 - A Real Mail Server, and a Genuinely Hard Debugging Session

M5 gave the panel real email: an account owner can create mailboxes, change their passwords, and
set up forwarders — mail sent to one address delivered to another, which doesn't even need to be
a mailbox on this server. On the admin side, a default mailbox quota setting and the mail
services showing up alongside everything else the server runs.

Under the hood this is Postfix (SMTP) and Dovecot (IMAP) — the same pairing real cPanel itself
uses — configured so both read mailbox, domain, and forwarder data directly out of the panel's
own database, live, through a database user that can only ever `SELECT` from exactly the three
tables it needs. No separate mail database to keep in sync with the app; the app's own records
are the only copy of the truth.

## The hard part

Postfix behaved. Dovecot did not, at least not at first — and for a good, specific reason: the
version that shipped on this box is Dovecot 2.4, released only this year, and its configuration
format changed substantially from the 2.3-and-earlier syntax that essentially every tutorial
and Stack Overflow answer online still shows. Old-style flat `passdb { }` blocks became named
`passdb sql { }` blocks. `%u` became `%{user}`. A `sql_driver` setting that isn't obviously
required broke authentication silently when left out. None of this was discoverable by reading
existing guides — it took working through Dovecot's own debug logging, one failure at a time,
to get a mailbox to actually authenticate and receive mail correctly.

Three real bugs came out of that process:

- Dovecot's default mail delivery setting quietly strips the domain off an email address before
  looking it up — a reasonable default for old-school "one mailbox = one Unix user" setups, and
  exactly wrong for a system where multiple mailboxes on different domains are looked up by
  full address. Every delivery failed with "user doesn't exist" until this was overridden.
- A leftover default meant for the old "single mailbox file per user" mail format was still
  active after switching to the modern per-message format, silently pointing new mail at the
  wrong, permission-denied location — even though every other piece of the per-mailbox path
  resolution was already correct. Tracked down through Dovecot's own verbose debug log, one
  layer at a time.
- The setup script itself had a bug: a credentials file that only root can read was being
  checked for existence by a user who isn't root, which silently reads as "the file doesn't
  exist" every time rather than raising a permission error — so the script kept generating a
  fresh password on every run without ever updating the actual database user to match, leaving
  the mail server pointed at a password that no longer worked. Caught by actually re-running the
  setup script against the live server a second time and confirming mail still delivered
  correctly, rather than trusting that it looked right on paper.

## What "done" actually meant here

Given this is a live mail server reachable from the entire internet, "it looks configured
correctly" wasn't good enough. Before calling this finished:

- A real message was sent by SMTP and landed in a real mailbox, on disk, exactly where it should.
- That mailbox was read back over a real encrypted IMAP connection with the real password.
- A forwarder was created and a message sent to it actually arrived at the other address.
- Authenticated outbound mail (the kind a real mail client sends) was tested and worked.
- And most importantly: connecting from a genuinely external IP address and trying to make the
  server relay mail to an unrelated domain was met with an outright refusal — confirmation this
  isn't an open relay, the single most common way a freshly-set-up mail server turns into
  something that gets the whole IP blacklisted within a day.

## Next

M6 — DNS. The Main server gets its own master nameserver; the account-side DNS zone editor lands
here too. DNS-only slave servers (the multi-server clustering piece) get scoped out during this
milestone but deliberately not built yet — that's an add-on to come back to once the core panel
itself is further along.
