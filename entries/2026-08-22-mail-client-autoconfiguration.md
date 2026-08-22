# Phase 121 - Mail client autoconfiguration

Second item off the "genuinely zero" list. cPanel's own "Mail Client Configuration" tools let a
mailbox owner point Thunderbird/Outlook/a phone mail app at their account with a domain and password —
no manually typing IMAP/SMTP hostnames and ports. This closes the same gap, scoped to what this
project's mail stack actually runs.

## Two halves, one toggle

**RFC 6186 SRV records** — `_imaps._tcp` and `_submission._tcp`, added to every new zone's defaults
alongside NS/A/MX (same toggleable pattern as everything else in Zone Templates:
`dns.template_include_mail_srv`, default on). The target hostname comes from
`MailServerHostnameResolver` — the same lookup KNJ Webmail's own IMAP/SMTP connections already go
through, so this respects Mail Only satellite linking for free: if a Mail Only server is active, the
SRV records point there; otherwise they point at this box.

**Mozilla ISPDB autoconfig XML** — served at the modern `/.well-known/autoconfig/mail/config-v1.1.xml`
path (plus the legacy `/mail/config-v1.1.xml`, still checked by some clients) directly on the customer's
own domain — no separate `autoconfig.<domain>` subdomain needed, since this is just a path on a vhost
that already exists. New `MailAutoconfigService` builds the XML and hands it to nginx via a `return
200 '...'` directive inside a per-account snippet, generated at account-provisioning time through the
exact same `write_account_vhost_snippet` mechanism `ErrorPageService`/`MimeTypesService` already use —
a new `autoconfig-write` provisioning action, but genuinely zero changes to any of the 5 vhost-creation
code paths, since every vhost already has the anchor line that mechanism attaches to.

## Only publishing what actually exists

This mail stack only has two client-facing ports actually open: IMAPS (993) and STARTTLS submission
(587) — confirmed against `deploy/mail/setup-mail-server.sh` (plain IMAP 143 is explicitly disabled)
and `FirewallService`'s `mail` port group (`25, 587, 993` — no 465, no 995, no 110). POP3 is installed
but never configured or firewalled. So neither the SRV records nor the autoconfig XML mention POP3 or
SMTPS at all — publishing DNS/autodiscovery for a service nothing answers on would just hand a client a
broken fallback, the same reasoning `DnsZoneService`'s own docblock already applies to webdisk and
CalDAV/CardDAV.

## Deliberately out of scope this pass

cPanel additionally serves the classic `autoconfig.<domain>` subdomain and Microsoft's Autodiscover
POX XML schema for Outlook. Both skipped: the subdomain fallback needs genuinely new infrastructure
(this project's `webmail`/`account`/`controller` shortcuts are static 301 redirects written once at
hostname-issue time, not per-domain dynamic content — serving different XML per domain from a shared
regex catch-all vhost is a different, larger mechanism), and Autodiscover is a separate, larger schema
with its own client-compatibility surface. Written up in `docs/roadmap.md` rather than silently skipped.

## Verified

4 new tests (2 `MailAutoconfigService` — snippet content, provisioning-script failure surfaces as an
exception; 2 `DnsZoneService` — SRV records backfilled by `restoreDefaultRecords()`, and the
`dns.template_include_mail_srv` toggle actually stopping them from being written) plus SRV assertions
added to the existing new-zone-creation test, and `ProvisionAccountJobTest`'s DNS record count updated
(12 → 14). Full suite 1,735 (up from 1,731), `pint --test` green.

## Live-verify addendum (same day) — found a real, pre-existing DNS bug

Shipped as v0.16.52, then live-verified against `test.knj.network`: backfilled the SRV records,
synced the autoconfig XML, and pulled both back with real DNS queries and a real HTTPS request.

The autoconfig XML was correct on the first try — real hostname, real ports, both paths serving 200.
The SRV records were not: `dig SRV _imaps._tcp.test.knj.network` returned
`mail.dev.knj.network.test.knj.network.` instead of `mail.dev.knj.network.` — the mail server's own
domain, with the customer's zone silently appended to the end.

Root cause, found by reading the raw zone file BIND actually wrote: `DnsZoneService::recordLine()`
absolutizes (adds the trailing dot) CNAME/MX/NS targets before writing them, so BIND treats them as
fully-qualified — but SRV was never included in that list, so its target was written bare. Per DNS
syntax, a name with no trailing dot is relative to the zone's own `$ORIGIN`; BIND correctly (per spec)
appended `test.knj.network` to a target that was never supposed to be relative in the first place.

This bug predates this feature entirely — any SRV record ever manually added through the Zone Manager
UI with an external target has always had it. It just took the panel generating its own SRV records
for the first time, pointing at a genuinely different domain (the mail server's hostname, essentially
always not the same as the customer's own domain), to actually exercise the broken path and surface
it. Fixed with a new `absolutizeSrvTarget()` that splits SRV's `weight port target` value and
absolutizes only the trailing target field, mirroring the same `absolute()` helper CNAME/MX/NS already
use. New regression test (`test_exported_zone_file_absolutizes_srv_targets`) covers a cross-domain
target directly. Shipped as v0.16.53.
