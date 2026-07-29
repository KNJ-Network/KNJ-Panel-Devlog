# Phase 29 - Contacts, and the Original Scope Is Now Complete

v0.7.0: contacts. A page under the sidebar to save a name and email address, edit or delete
either inline, no page reload needed for either. Two small conveniences tie it into the rest
of the client rather than leaving it as an island: saved contacts back an autocomplete list
on compose's "To" field, and reading any message offers a one-click "+ Contact" button that
saves its sender without leaving the message.

That button is the one piece worth a note. It parses the sender straight from the message's
raw `From` header, which shows up in two shapes depending on what sent it — `Name
<email@example.com>` from a real mail client, or just a bare address from something more
basic — so the parsing has to handle both, falling back to using the address as the name
when there isn't one. And because that button lives on a page with nowhere to put a
validation error, adding a sender who's already saved is a silent no-op rather than a
failed form submission bouncing the reader off the message they were reading.

With this, everything in the original webmail scope — inbox, folders, rules, rich compose,
bulk actions, contacts — is shipped.

Verified by hand on the real mailbox: added a contact directly, confirmed it showed up in
compose's autocomplete, then opened a genuine Gmail-sent message and used the real "+
Contact" button to save its sender — correctly parsed from an actual `Name <email>` header,
not a synthetic test one — clicked it a second time to confirm the duplicate silently
no-ops instead of erroring, then edited and deleted both contacts through the real UI.
