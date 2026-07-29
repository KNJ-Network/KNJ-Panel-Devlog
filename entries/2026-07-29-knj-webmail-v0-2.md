# Phase 24 - KNJ Webmail Starts Looking Like a Real Mail Client

The foundation from last time (login, inbox, read, compose, send) gets the batch of features
that actually make it usable day to day — v0.2.0.

Real folders now: Inbox, Sent, Drafts, Trash, Spam, plus anything a user creates themselves,
with unread counts in the sidebar. That needed a small infrastructure fix underneath —
Dovecot ships those special-use folders defined but not auto-created, so they only used to
exist once some client (Roundcube, say) created them on demand. Every mailbox gets them from
day one now.

Delete moves a message to Trash rather than permanently expunging it, matching what every
real mail client does — Trash itself is where "delete forever" actually applies. Sending a
message now files a copy into Sent via IMAP too; previously it went out fine but was never
filed anywhere, which isn't how a real client behaves. Attachments are listed with size and
downloadable on a message, and compose gained upload support, sent via Symfony Mailer.
Rounding it out: search within the open folder, pagination past the first page, and saving to
Drafts with a way to reopen a draft back into the compose form.

A few things only showed up under real use, not in review:

- The delete confirmation was the browser's own native popup — jarring next to a UI that's
  otherwise entirely its own. Replaced with an in-app modal.
- "Save draft" was blocked by the same required-field validation meant for the Send button —
  a half-finished draft should be saveable.
- The signed-in mailbox address was buried in small print at the bottom of the sidebar;
  moved it to sit clearly under the logo instead, and the sidebar's footer now shows the
  webmail client's own version number, tracked independently of the panel's version — a
  panel release with no webmail changes doesn't bump it, and a webmail-only update doesn't
  bump the panel's.

Verified the same way as last time, not just against the test suite: real login, a real
folder move, a real draft saved and reopened, and a real send — confirmed via the receiving
server's own log, not just the outbound queue.

Still ahead: bulk actions on multiple messages at once, richer (HTML) compose, contacts.
