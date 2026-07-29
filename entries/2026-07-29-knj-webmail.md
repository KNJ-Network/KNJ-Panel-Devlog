# Phase 23 - Our Own Webmail Client, Not Just a Reskin

The webmail client the panel was built ahead of last time: KNJ Webmail, built into the panel
itself rather than a separate app. Same layout, same colors, same logo — a second, selectable
option on the Email Accounts page. Sign-in is its own login, checked against the mailbox's IMAP
credentials, separate from the panel's own user accounts.

IMAP is handled through a pure-PHP library. That surfaced a real bug worth writing down: encoded
subject and sender headers showed up in raw form for messages from some senders instead of the
actual text. Fixed by decoding them ourselves.

Covers the essentials: inbox, reading a message, replying, composing, deleting. Message bodies are
rendered in an isolated frame, since email HTML is untrusted content.

Verified against a test mailbox on a development server: signed in through the KNJ Webmail login,
read a real message, sent a new one, and confirmed from the receiving server's own log that it was
delivered rather than bounced or filtered.
