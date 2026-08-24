# Phase 129 - CalDAV/CardDAV (Calendars & Contacts Sync)

A custom SabreDAV integration built into the panel itself, for the same reasons as building KNJ
Webmail: full control and no second credential store. It was the most self-contained of the three
features scoped together (with the WAF and app hosting): it needed no new provisioning actions and
no new services.

## What shipped

Every mailbox gets a default calendar and address book when created, so a client that only
auto-discovers has something to sync against from its first connection. Authentication reuses the
mailbox password already used for IMAP, SMTP and webmail, so there is one password everywhere and no
new credential store.

The service is reachable on the account port under `/dav/`, with each request carrying its own
authentication. RFC 6764 discovery is wired up: CalDAV/CardDAV SRV records (pointing at Main, since
DAV data lives only in Main's database) and well-known redirects, both toggleable from Zone Templates.
It is limited to Main, since satellites are separate installs with separate databases.

There is deliberately no calendar or contact browser in the account panel; a DAV client is the
browser. The one UI surface is a read-only connection-info page per mailbox with the server URLs for
clients that need manual entry.

## Real bugs, caught by tests and live checks

Two were caught by writing tests: table names inferred by the framework did not match the migration
for the address-book models, and a constant referenced by the address-book backend exists only on the
CalDAV plugin class.

A third was found live: a request to the bare DAV root, exactly what a real client sends first, was
rejected because the exemption pattern did not match the root path itself. It was fixed by adding the
root alongside the wildcard.

## Verified

Live on the dev server against a real disposable mailbox: root, calendar-home and address-book-home
requests succeeded; calendar creation, event upload and download round-tripped byte-for-byte; contact
upload did the same; deletion and well-known redirects behaved correctly; and discovery records
resolved correctly. All test state was cleaned up.

25 new tests with the full suite green (1907/1907) and the code-style check clean. Sharing between
mailboxes and scheduling invitations were deliberately left out of scope.
