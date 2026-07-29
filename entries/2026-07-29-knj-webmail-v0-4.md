# Phase 26 - Rules, and a Settings Page to Hold Them

v0.4.0: a Settings page, reached from the bottom of the sidebar, covering the per-mailbox
autoresponder and a new Outlook-style rules feature — "if From/To/Subject contains or is a
value, file the message into folder X," tried in order, first match wins.

The interesting part is underneath, not the form itself. The autoresponder already worked by
writing a Sieve script to the mailbox — Pigeonhole, Dovecot's Sieve engine, was already
installed and already ran it. Adding rules on top meant that one mailbox's script now needs to
hold two different features' worth of logic at once, so instead of the autoresponder alone
owning that file, there's now one shared builder and one shared writer both features go
through. Vacation always comes first in the generated script, since a rule's `stop;` would
otherwise stop it from ever running — filing and auto-replying both happen, in the right
order, on the same message.

Also in this batch, both straight from feedback on the previous release: the Date column
moved from the far right of the message list to the left, and all three columns are now
sortable by clicking the header. Date sorts properly across an entire folder, since the mail
server orders messages by arrival natively — From and Subject only sort within whatever page
is currently on screen, since plain IMAP has no equivalent for arbitrary header fields without
reaching for a protocol extension the mail server doesn't run. Worth knowing, not a bug.

Verified the same way as every batch before it: a real rule, added through the actual Settings
page, confirmed by reading the exact Sieve script it wrote on the mail server — then confirmed
the script was removed again once the rule was deleted.
