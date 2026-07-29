# Phase 24 - KNJ Webmail Starts Looking Like a Real Mail Client

The foundation from last time (login, inbox, read, compose, send) gets the batch of features that
make it usable day to day — v0.2.0.

Real folders now: Inbox, Sent, Drafts, Trash, Spam, plus any a user creates, with unread counts in
the sidebar. That needed a small infrastructure fix underneath: the mail server defines those
special folders but doesn't create them automatically, so they only existed once some client
created them. Every mailbox gets them from day one now.

Delete moves a message to Trash rather than permanently removing it; Trash is where "delete
forever" applies. Sending a message now files a copy into Sent. Attachments are listed with size
and downloadable, and compose gained upload support. Also: search within the open folder,
pagination, and saving to Drafts with a way to reopen a draft.

A few things only showed up under real use:

- The delete confirmation was the browser's native popup, which looked out of place. Replaced with
  an in-app modal.
- "Save draft" was blocked by the validation meant for Send. A half-finished draft should be
  saveable.
- The signed-in mailbox address was in small print at the bottom of the sidebar; it now sits under
  the logo, and the footer shows the webmail client's own version number, tracked independently of
  the panel's.

Verified against a development server: real login, a folder move, a draft saved and reopened, and
a real send, confirmed via the receiving server's log.
