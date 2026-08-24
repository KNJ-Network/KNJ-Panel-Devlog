# Phase 129 - CalDAV/CardDAV (Calendars & Contacts Sync)

A custom SabreDAV integration built into the panel itself, same reasoning as building KNJ Webmail
instead of relying solely on Roundcube: full control, no second credential store. This turned out to
be the most self-contained of the three features scoped alongside it (WAF and app hosting) — despite
sounding like a big mail-adjacent subsystem, it needed zero new provisioning-script actions and zero
new systemd units. It's plain Laravel plus a new Composer dependency plus two new nginx lines.

## What shipped

Every mailbox gets one default calendar and one default address book the moment it's created, seeded
by the same `MailboxService::createMailbox()` path every other mailbox feature already hooks into —
a client that only ever auto-discovers, never sends its own `MKCALENDAR`, needs something real to
sync against from the first connection. Authentication reuses the exact SHA512-CRYPT hash already
stored for IMAP/SMTP/webmail (`{SHA512-CRYPT}` stripped, handed to PHP's own `crypt()`), so this adds
no new credential store at all — one password everywhere, exactly like the rest of this mail stack.

The whole thing is reachable at the existing account port under `/dav/`, outside the panel's session
system entirely: every DAV request carries its own HTTP Basic Auth, so there's no session, no CSRF
token, and no `auth` middleware group to sit inside. RFC 6764 discovery is fully wired — `_caldavs.
_tcp`/`_carddavs._tcp` SRV records (pointing at Main's own hostname, deliberately never a linked Mail
Only satellite, since DAV data lives only in Main's own database) and `.well-known/caldav`/`carddav`
redirects, both toggleable from Zone Templates alongside the existing mail-autodiscovery SRV
records. Gated Main-only via a dedicated `EnsureCanAccessDav` middleware, for the same reason as the
SRV-record targeting: a DNS-only or Mail Only satellite is a fully separate Laravel install with its
own separate database, and there's no mechanism for Main to query one live.

Deliberately no calendar/contact data browser in the account panel — a DAV client is the browser. The
one UI surface is a minimal read-only connection-info page per mailbox: server URL, explicit
CalDAV/CardDAV URLs for clients that need manual entry (Outlook, some Android apps), and a reminder
that the password is the same one already in use everywhere else.

## Two real bugs, caught by writing tests, not from documentation

**Eloquent's inferred table names didn't match the migration.** `DavAddressBook`,
`DavAddressBookObject`, and `DavAddressBookChange` had no explicit `$table` property, so Eloquent's
default snake-case-and-pluralize inference gave `dav_address_books` (three words) — but the
migration follows SabreDAV's own terminology of "addressbook" as one word, matching
`dav_addressbook_objects`/`dav_addressbook_changes`. Every query against those three models would
have failed in production with a real "no such table" error the moment anything touched contacts;
the calendar side was unaffected since "calendar" has no such compound-word ambiguity. Caught the
first time a real test tried to insert a row.

**A constant that only exists on the other plugin class.** `AddressBookBackend` referenced
`Sabre\CardDAV\Plugin::NS_CALENDARSERVER` for the `getctag` property — a constant that turns out to
only actually be defined on `Sabre\CalDAV\Plugin`, even though `getctag` lives under the
"calendarserver" namespace for CardDAV collections too. `CardDAV\Plugin` has no equivalent of its
own. A straight `Undefined constant` fatal, also caught the moment a real test exercised the address
book listing path.

## The bug that made every real CalDAV client's first request fail

Found live, not in `php artisan test` — every raw-protocol check against a nested path (like
`/dav/calendars/<address>/`) worked fine, but a `PROPFIND` against the bare DAV root (`/dav/`, no
trailing content) came back Laravel's own `419 Page Expired` instead of reaching the DAV server at
all. That root-level request is exactly what a real client sends first, before it even knows any
calendar/address-book paths exist.

Root cause: `'dav/*'` in the CSRF exemption list never matches a request to exactly `/dav/` or
`/dav`. `Illuminate\Http\Request::path()` strips the trailing slash before pattern matching runs, so
the trimmed path `dav` has no `/` left for the wildcard to follow — confirmed directly with a one-line
reproduction (`Request::create(".../dav/", "PROPFIND")->is('dav/*')` returns `false`, `->is('dav')`
returns `true`). Nested paths were never affected, which is exactly why this only surfaced once a
genuine root-level request was tried. Fixed by adding the bare `'dav'` entry alongside `'dav/*'`.

## Verified

Live, end to end, on `panel-dev`, against a real disposable mailbox created through the real
provisioning path (not a raw DB insert). Real `PROPFIND` against the DAV root, the calendar home,
and the address book home all returned `207`. A real `MKCALENDAR` against a brand-new calendar
returned `201`. A real `.ics` event `PUT`, then `GET` back and diffed byte-for-byte identical to what
was sent; the same round-trip for a real `.vcf` contact card. `DELETE` returned `204`, and a
follow-up `GET` for the same object correctly came back `404`. `.well-known/caldav` and
`.well-known/carddav` both `301`'d to the correct account-port URL. `dig` against Main's own
nameserver confirmed real `_caldavs._tcp`/`_carddavs._tcp` SRV records resolving to the right host
and port, backfilled onto the pre-existing test zone via the same `restoreDefaultRecords()` path
already used for mail-autodiscovery backfills. All test state (the disposable mailbox and its
cascade-deleted default calendar/address book) cleaned up afterward.

25 new unit/feature tests, full existing suite green (1907/1907), `pint --test` clean. Deferred
non-goals stated explicitly, not guessed at: sharing/delegation between mailboxes, and CalDAV
scheduling (iTIP free/busy, meeting invites) — real complexity with zero existing precedent in this
codebase to build on.
