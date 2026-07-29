# Phase 23 - Our Own Webmail Client, Not Just a Reskin

The webmail client the panel was built ahead of last time: KNJ Webmail, built into the panel
itself rather than a separate app like Roundcube. Same layout, same colors, same logo — a
second, selectable option on the Email Accounts page, not a bolt-on. Sign-in is its own
session, checked directly against the mailbox's IMAP login, entirely separate from the panel's
own user accounts.

IMAP is handled through a pure-PHP library rather than PHP's own IMAP extension, which the
servers deliberately don't ship. That choice surfaced a real bug worth writing down: without
the extension present, the library silently skips decoding MIME-encoded subject and sender
headers — the raw `=?UTF-8?Q?...?=` form showed up untouched for messages from real senders
(Gmail included), rather than the actual text. Fixed by decoding it ourselves instead of
relying on the library to.

Covers the essentials: inbox, reading a message, replying, composing, deleting. Message bodies
render inside a sandboxed iframe with scripts and forms switched off — email HTML is untrusted
content, not something to trust with the same page.

Verified against the same real, publicly delegated test mailbox proven for Roundcube: signed in
through the actual KNJ Webmail login, read a real message, sent a new one, and confirmed via the
receiving server's own log — not just our own outbound queue — that it was delivered rather than
bounced or filtered.
