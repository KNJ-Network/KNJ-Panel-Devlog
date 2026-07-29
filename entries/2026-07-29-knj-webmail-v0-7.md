# Phase 29 - Contacts, and the Original Scope Is Now Complete

v0.7.0: contacts. A page under the sidebar to save a name and email address, edit or delete either
inline, no page reload needed. Two conveniences tie it into the rest of the client: saved contacts
back an autocomplete list on compose's "To" field, and reading any message offers a one-click
"+ Contact" button that saves its sender without leaving the message.

That button has to handle senders that appear as `Name <email@example.com>` or as a bare address,
falling back to the address as the name when there isn't one. Adding a sender who's already saved
is a silent no-op rather than an error that would bounce the reader off the message.

With this, everything in the original webmail scope — inbox, folders, rules, rich compose, bulk
actions, contacts — is shipped.

Verified by hand on a test mailbox: added a contact, confirmed it appeared in compose's
autocomplete, used "+ Contact" on a real received message, confirmed a duplicate was a silent
no-op, then edited and deleted both contacts.
