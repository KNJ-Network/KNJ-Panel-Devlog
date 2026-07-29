# Phase 26 - Rules, and a Settings Page to Hold Them

v0.4.0: a Settings page, reached from the bottom of the sidebar, covering the per-mailbox
autoresponder and a new Outlook-style rules feature — "if From/To/Subject contains or is a
value, file the message into folder X," tried in order, first match wins.

The interesting part is underneath. The autoresponder already worked through the mail server's
standard filtering engine. Adding rules meant one mailbox's filter needed to hold two features'
worth of logic, so both features now go through one shared builder, with ordering handled so that
filing and auto-replying both happen on the same message.

Also in this batch, both from feedback on the previous release: the Date column moved from the far
right of the message list to the left, and all three columns are now sortable by clicking the
header. Date sorts across an entire folder; From and Subject sort within the current page, since
plain IMAP has no equivalent for arbitrary header fields.

Verified with a real rule added through the Settings page, checked on the mail server, then
confirmed removed again once the rule was deleted.
