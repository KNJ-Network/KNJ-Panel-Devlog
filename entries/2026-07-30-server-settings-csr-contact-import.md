# Phase 31 - Resolver Config, Server Contacts, Reboot, CSR Generator, Contact Import

Five smaller items boxed off today, rounding out Server Settings and closing a couple of
long-standing gaps on the SSL and Webmail Contacts pages.

**Server Settings** picked up three new cards alongside yesterday's timezone and hostname:
Resolver Configuration (the upstream DNS resolvers this box itself uses for outbound lookups,
applied through `systemd-resolved`, separate from the zones it serves as a nameserver), Server
Contacts (who gets emailed about pending updates or SSL issuance failures), and a Reboot Server
danger zone that schedules a graceful `shutdown -r +1` so the confirmation actually reaches the
browser before the box goes down. The page went from one long column of identical white cards to
a proper two-column grid, and a checkbox row that had gone edge-to-edge with the border got some
actual padding.

**Generate a Certificate Signing Request** landed on the account SSL page — a fresh 2048-bit
private key and CSR generated entirely in-process through PHP's own openssl extension, no shell-out,
no key material ever written to disk. Shown once with a clear "copy this now" warning, since nothing
is stored server-side.

**Bulk CSV import** for Webmail Contacts rounds out the contacts feature from earlier this week —
upload a CSV of either `name,email` pairs or bare email addresses, header row auto-detected,
invalid or already-saved rows skipped and counted back to you.

One real bug caught along the way: the timezone dropdown was silently defaulting to whatever
sorted first alphabetically, because Ubuntu reports the system timezone as `Etc/UTC` and PHP's
default `DateTimeZone::listIdentifiers()` doesn't include the `Etc/*` set — switched to
`ALL_WITH_BC` so the current value is always a real option in the list.

Also: CI had been quietly red since yesterday's Server Settings work landed — a Pint code-style
check, not caught locally before pushing. Fixed and confirmed green before anything else shipped
today.

Also caught and fixed in passing: KNJ Webmail's sign-in page had its own branding backwards — the
top-left header said "Sign in" and the login card itself said "KNJ Webmail", swapped the two so
the header matches every other page and the card says what it's actually for. And in the signed-in
sidebar, the current mailbox address and version number were left-aligned under the centered
logo/title — centered both to match.

Verified live on the real dev server: timezone/hostname/resolvers/contacts all save and persist,
reboot renders correctly with its confirm dialog, the CSR generator produces a real, well-formed
certificate request and key, and the contact import was verified with real feature tests (valid
rows imported, duplicates and invalid rows skipped, header detection) since a file-upload dialog
isn't something a browser automation session can drive directly.
