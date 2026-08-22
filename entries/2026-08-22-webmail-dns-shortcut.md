# Phase 118 - webmail.domain and domain.com/webmail, cPanel-style

Last item picked up today: closing out a gap flagged and deliberately deferred back on 2026-07-25 —
`docs/roadmap.md` and `DnsZoneService`'s own docblock both listed `webmail.<domain>` as one of several
service-discovery records "deliberately not done," on the reasoning that publishing DNS for a service
nothing answers on is worse than not publishing it. That was correct at the time: KNJ Webmail didn't
exist yet. It's been fully built and shipped since (v0.7.0, 2026-07-29) and nobody went back to close
the loop. Today's DNSONLY comparison audit surfaced the stale deferral while researching cPanel's own
webmail-access conventions, which is what prompted actually building it.

## What cPanel does, and what this ships

Real cPanel gives every hosting customer two interchangeable ways into webmail: `webmail.domain.com`
(a subdomain) and `domain.com/webmail` (a path on the customer's own site). Both now work here too.

**The DNS record** — `webmail` A record, same toggleable pattern as the existing `mail` record
(`dns.template_include_webmail`, defaulting on), added to both `createZoneForSite()` (new zones) and
`restoreDefaultRecords()` (the self-service "Restore Default Records" recovery action).

**`domain.com/webmail`** — every customer vhost already `include`s one shared Nginx snippet
(`NginxSettingsService`, server-context, driven by Controller → Nginx Settings). Added one
`location /webmail { return 301 https://<panel-hostname>:2083/webmail; }` line there instead of
touching the half-dozen separate vhost-creation code paths in the provisioning script individually —
one change, live on every existing domain immediately, and every domain created afterward for free.
A redirect, not a reverse proxy: the target already has valid HTTPS on the panel's own hostname, so
there's no TLS/SNI mismatch or session-cookie-domain awkwardness to work around. Omitted entirely
until a panel hostname actually exists (a fresh install's landing-page vhost, pre–Server Setup) —
redirecting to `https://:2083/webmail` would just be broken.

**`webmail.domain.com`** — no per-customer vhost has a matching `server_name`, so this needed its own
piece: one regex-`server_name` (`~^webmail\.(?<webmail_domain>.+)$`) catch-all vhost, written by the
same `issue-panel-cert` provisioning-script action that already (re)writes the panel's own vhost on
initial setup and on every later hostname change — so the redirect target can never go stale the way
a bootstrap-time-only value would. HTTP-only (port 80) deliberately: a customer domain's own TLS cert
only ever covers that exact domain, never `webmail.<that domain>` too, so terminating HTTPS here would
present the wrong certificate before any redirect could fire. nginx only reaches this regex match when
no more-specific vhost exists for that name — real customer vhosts always register an exact
`server_name` match, which nginx tries first — so a customer whose own primary domain happens to
literally start with "webmail." is never intercepted by this catch-all. Role-gated to `main` only:
DNS Only and Mail Only satellites host no customer domains, so there'd be nothing for it to ever
redirect from.

**Kept in sync on hostname change** — `PanelCertService::issue()` now also calls
`NginxSettingsService::apply()` right after setting `panel.hostname`, so every existing customer
vhost's `/webmail` redirect target updates immediately on a hostname change too, rather than staying
broken until an admin happens to separately re-save Nginx Settings.

## What's still correctly deferred

cPanel/WHM, WebDisk, autodiscover/autoconfig XML endpoints, and CalDAV/CardDAV SRV+TXT records remain
undone — those genuinely point at services this install still doesn't run. `webmail` was the one item
on that original list whose blocker had actually cleared.

## Verified

Full local suite (1,708 tests, up from 1,703) and `pint --test` green, plus `bash -n` on the
provisioning script. New coverage: `DnsZoneServiceTest` (webmail record on create + on
`restoreDefaultRecords()` backfill), `DnsZoneTemplateTest` (toggle-off stops it being written),
`NginxSettingsTest` (the redirect line appears with a hostname set, is omitted without one), and
`ServerSetupTest` (issuing the panel cert also fires `nginx-settings-write`, proving the redirect
target actually gets regenerated).
