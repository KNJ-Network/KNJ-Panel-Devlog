# Phase 08 - A Real Mail Server, and a Genuinely Hard Debugging Session

M5 gave the panel real email: an account owner can create mailboxes, change their passwords, and
set up forwarders — mail sent to one address delivered to another, which doesn't even need to be
a mailbox on this server. On the admin side, a default mailbox quota setting and the mail
services showing up alongside everything else the server runs.

Under the hood this is Postfix (SMTP) and Dovecot (IMAP) — a proven, widely-used pairing —
configured to read mailbox, domain, and forwarder data directly from the panel's own database,
using a restricted read-only database user. No separate mail database to keep in sync; the app's
own records are the only copy of the truth.

## The hard part

Postfix behaved. Dovecot did not, at least not at first — the version on this box is Dovecot 2.4,
and its configuration format changed substantially from the 2.3-and-earlier syntax that most
tutorials still show. None of this was discoverable by reading existing guides — it took working
through Dovecot's own debug logging, one failure at a time, to get a mailbox to authenticate and
receive mail correctly.

Three real bugs came out of that process:

- A default in how recipient addresses are looked up was wrong for a system hosting several
  domains, so deliveries failed until it was overridden.
- A leftover default from an older mailbox format was still active after switching to the modern
  one, pointing new mail at the wrong location.
- The setup script mishandled a root-only credentials file when run as a non-root user, so
  re-running it could leave the mail server and database out of step. Caught by re-running the
  setup against a live development server and confirming mail still delivered.

## What "done" actually meant here

Because this is a mail server reachable from the internet, "it looks configured correctly" wasn't
good enough. Before calling this finished:

- A real message was sent by SMTP and landed in a real mailbox.
- That mailbox was read back over an encrypted IMAP connection.
- A forwarder was created and a message sent to it arrived at the other address.
- Authenticated outbound mail (the kind a real mail client sends) was tested and worked.
- Connecting from an external IP and trying to relay mail to an unrelated domain was refused,
  confirming the server isn't an open relay.
