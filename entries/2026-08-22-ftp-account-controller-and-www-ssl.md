# Phase 119 - ftp/account/controller shortcuts, and closing the www SSL gap

Follow-up to yesterday's webmail DNS-shortcut work, picked up directly from the DNSONLY comparison
audit's own findings: `ftp.<domain>` and cPanel's `cpanel.<domain>`/`whm.<hostname>` shortcuts were the
next two candidates on that same list, and a real live SSL bug got found and closed alongside them.

## ftp.<domain>

Smallest possible version of this pattern — FTP is fully built and already live-verified, so this is
purely the DNS record (`dns.template_include_ftp`, same toggleable shape as `mail`/`webmail`). No
nginx work at all, since FTP isn't HTTP — there's no path or subdomain redirect to build.

## account.<domain> and controller.<domain>

cPanel's own `cpanel.<domain>` (account panel) and `whm.<hostname>` (admin panel) shortcuts, renamed
since this project can't use either trademarked name. Same three-piece mechanism as yesterday's
webmail fix:

- `dns.template_include_account` / `dns.template_include_controller` — A records, same toggle pattern.
- `location /account` / `location /controller` in `NginxSettingsService`'s shared vhost snippet —
  redirects to `https://<panel-hostname>:2083/account` and `:2087/controller` respectively.
- Two more regex-`server_name` catch-all vhosts in `issue-panel-cert`, alongside the existing webmail
  one — `account.*` → :2083, `controller.*` → :2087.

## The www SSL gap — a real live bug, not a missing convenience

While researching all of this, a genuine bug turned up: every account's own vhost has always listed
`www.<domain>` in its `server_name` — that's not new — but `AccountProvisioningService::issueSsl()`
only ever requested the apex domain as the certificate's SAN. `www.<domain>` was being served by the
exact same vhost, just with a certificate that never covered it. Every visitor to
`https://www.<domain>` on every account has been hitting a real certificate-mismatch warning.

A real cPanel screenshot (via trycpanel.net's SSL/TLS page) confirmed what the fix should be:
`example.com` and `www.example.com` share one certificate — a single green padlock, not two. Fixed by
requesting both names in the one `certbot --nginx` call. Multi-name issuance is all-or-nothing, so if
`www` isn't actually resolving to this server yet (record deleted, still propagating, pointed
elsewhere), the whole request would otherwise fail — added a fallback that retries apex-only in that
case, so an account never ends up with *no* certificate over one unreachable name.

## Deliberately not done this pass: bundling mail/webmail/account into that same certificate

cPanel's own screenshot shows one certificate covering `example.com`, `www.example.com`,
`mail.example.com`, `webdisk.example.com`, `webmail.example.com`, and `cpanel.example.com` all
together — and, tellingly, explicitly *excludes* `whm.example.com` from that bundle (shown with a red,
uncovered padlock), since that's a server-wide surface, not an account one. This project's own
`controller.<domain>` (the `whm.` equivalent) should stay excluded the same way.

Extending this pass to bundle `mail`/`webmail`/`account.<domain>` into the account's own certificate
too — closer to full cPanel parity — was scoped out deliberately. Doing it safely means those hostnames
need to be real `server_name` entries on the customer's own vhost (not just the separate HTTP-only
catch-alls this and yesterday's webmail fix use), which means touching every vhost-creation code path
in the provisioning script — `create`, `add-domain`, `add-subdomain`, `convert-addon-domain` — each of
which currently generates slightly different vhost content. That's real, separate surface with real
risk to live customer HTTPS if rushed, not something to bundle into the same pass as three other
features and a live bug fix. The `www` fix here was safe to do immediately specifically because `www`
was *already* in every vhost's `server_name` — nothing else on this list can currently make that claim.

## Verified

Full local suite (1,710 tests, up from 1,703) and `pint --test` green, plus `bash -n` on the
provisioning script. New coverage: `DnsZoneServiceTest` (all three new records on create + on
`restoreDefaultRecords()` backfill), `DnsZoneTemplateTest` (toggling all three off stops them being
written), `NginxSettingsTest` (both new redirect lines appear/are omitted correctly), and
`ProvisionAccountJobTest`'s record-count assertion updated (9 → 12).

The `ssl` provisioning action's certbot logic has no automated test harness (same as every other
bash-only change this session) — verified by hand-tracing the `set -e`/`if ! cmd; then` fallback
pattern, matching the exact idiom already proven correct twice yesterday for the `issue-panel-cert`
certbot fixes.
