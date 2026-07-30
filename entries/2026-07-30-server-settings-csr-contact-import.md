# Phase 31 - Resolver Config, Server Contacts, Reboot, CSR Generator, Contact Import

Five smaller items boxed off today, rounding out Server Settings and closing a couple of gaps on
the SSL and Webmail Contacts pages.

**Server Settings** picked up three new cards alongside yesterday's timezone and hostname:
Resolver Configuration (the upstream DNS resolvers this server uses for outbound lookups, separate
from the zones it serves as a nameserver), Server Contacts (who gets emailed about pending updates
or SSL issuance failures), and a Reboot Server danger zone that schedules a graceful reboot shortly
after confirmation so the confirmation reaches the browser first. The page also moved from one long
column of identical cards to a two-column grid.

**Generate a Certificate Signing Request** landed on the account SSL page — a fresh private key and
CSR generated in-process by PHP's openssl extension. They are shown once with a clear "copy this
now" warning, since nothing is stored server-side.

**Bulk CSV import** for Webmail Contacts rounds out the contacts feature — upload a CSV of either
`name,email` pairs or bare email addresses, with the header row auto-detected and invalid or
already-saved rows skipped and counted.

One bug caught along the way: the timezone dropdown defaulted to whatever sorted first
alphabetically, because the system's current timezone wasn't in PHP's default list. It now uses the
extended list so the current value is always an option.

Also fixed in passing: KNJ Webmail's sign-in page had its header and card titles swapped, and the
signed-in sidebar's mailbox address and version were misaligned under the centered logo.

Verified on a development server: timezone, hostname, resolvers, and contacts all save and persist,
reboot shows its confirm dialog, the CSR generator produces a well-formed request and key, and the
contact import was verified with feature tests, since a file-upload dialog can't be driven by
automated test tooling.
