# Phase 117 - cPanel DNSONLY comparison audit

Last item on the day's list, after the mail wrap-up, feature/competitor audit, three security audits, and
the line-by-line bug sweep: an exhaustive page-by-page comparison of the real cPanel DNSONLY product
against our own `dns_only` role, to catch anything a genuine DNS-only nameserver operator would expect
that we've quietly left out.

No live DNSONLY trial instance exists to click through, so this pass worked from cPanel's own
documentation (docs.cpanel.net's DNSONLY and DNS Cluster Configuration pages) plus several independent
third-party overviews, cross-checked against our actual `App\Support\DnsOnlyRoleGate` allowlist and the
real controllers behind each candidate route — not just the generic feature list, since a route existing
doesn't mean it's actually relevant to a satellite with no hosting accounts.

## What was already covered

Most of cPanel's documented DNSONLY feature set turned out to already be reachable, mostly from earlier
Tier 1 parity work (2026-08-21):

- Hostname, resolver, and contact-email management — `Server Settings`
- Graceful/forceful reboot and per-service restarts — `Server Settings` + `Service Manager`
- DNS cluster configuration and record sync — `DNS Replication` + `Server Link` (our own two-key
  mutual-auth architecture doing the same job cPanel's API-token cluster trust relationship does)
- Log rotation, SSL Storage Manager, Security Policies, Diagnostics, System/Panel Updates
- Two-Factor Authentication — Fortify's own routes were never inside the `controller.` name group
  `DnsOnlyRoleGate` gates in the first place, so this was already unrestricted without anyone having to
  think about it

Correctly out of scope by design, not gaps: no support-ticket system (we don't have one at all), no
multi-IP management page (single-IP-per-box hosting model), no BIND/PowerDNS selector (we only ever
install BIND).

## The real gap

**Firewall, Access Control, and Password Policy were missing from both satellite roles' allowlists —
not just DNS-only.** All three are role-agnostic OS/login security pages, already built and working for
Main, that apply just as directly to a satellite's own SSH and panel logins:

- `security.firewall` — fail2ban/ufw status and unban, scoped to the `sshd` and `knjpanel-auth` jails.
  This is the exact feature cPanel documents by name as "cPHulk Brute Force Protection." Without it, an
  admin whose DNS-only box (port 53, wide open to the internet by definition) or Mail-only box (port 25)
  got a legitimate IP fail2ban-blocked had no way to see or clear that ban except SSH — directly against
  this panel's own design goal of never requiring operator SSH for day-to-day management.
- `security.access-control` — per-service IP allow/deny. `FirewallService::SERVICES` already lists a
  `dns` entry for port 53 tcp/udp specifically, and a `mail` entry for 25/587/993 — both exactly the
  ports each satellite role most needs to restrict, and both already coded, just unreachable.
- `security.password-policy` — governs `PasswordPolicyService::rule()`, which `Fortify\
  PasswordValidationRules` wires into the login/password-change pipeline itself. This isn't a
  hosting-account setting; it's the requirements for *this box's own admin login*, which a satellite
  admin has just as much reason to tighten as a Main admin does.

Fixed by adding all three to both `DnsOnlyRoleGate::ALLOWED_ROUTE_PREFIXES` and
`MailOnlyRoleGate::ALLOWED_ROUTE_PREFIXES`, plus matching nav entries in both roles' Security group in
`WhmController::navGroups()` — the allowlist alone would have made the pages reachable by direct URL but
left them invisible in the sidebar, the same "hidden from nav isn't the same as actually gated" distinction
`DnsOnlyRoleGate`'s own doc comment already calls out in the other direction.

## Confirmed *not* gaps — checked the code, not just cPanel's list

Two more candidates looked plausible from cPanel's generic feature list alone, and turned out to be
correctly excluded once the actual controllers were read:

- **API Token management** (`security.tokens`) — `ApiTokenController::ABILITIES` is exclusively
  `accounts:*` and `domains:*` scopes: create/suspend/terminate a hosting account, convert an addon
  domain. Every single ability is a Main-only hosting concept with no equivalent on a satellite that has
  no accounts at all. Exposing this page would let a DNS-only admin mint a token whose only possible
  abilities all 404 or no-op on that box.
- **DNS zone TTL/SOA defaults** (`services.dns`, `DnsSettingsController`) — its own update() message says
  it plainly: "applied to new zones." Satellites never originate zones; they only ever receive zones
  pushed from Main. Exposing this on a DNS-only box would show working-looking settings that silently do
  nothing there.

## Flagged, not fixed

**Service Certificates** (`services.certificates`, read-only visibility for which services share the
panel's one Let's Encrypt cert) is a real, legitimate gap — cPanel documents "Service SSL certificate
management" by name — but `ServiceCertificatesController::SHARED_SERVICES` is a flat, role-blind constant:
`['Panel...', 'Postfix...', 'Dovecot...']`. Exposing the page as-is to `dns_only` would tell the admin
their (nonexistent) Postfix and Dovecot share the panel's cert — worse than not having the page at all.
Needs the same kind of role-aware list `ServiceStatusService::MAIL_ONLY_EXCLUDED` already established in
today's bug sweep before it can be added, not a same-day allowlist-only fix. Noted for a future pass.

## Verified

Full local suite (1,703 tests, up from 1,701) and `pint --test` green. Added dedicated coverage in both
`DnsOnlyRoleGateTest` and `MailOnlyRoleGateTest` proving all three new routes return 200 under their
respective role, matching the existing Tier-1-parity test pattern in both files.

Deployed as v0.16.44 to all three servers, version-confirmed on all three, then hit all three new routes
for real against both live-linked satellites (not just Main, where they already worked) — six checks, all
200: `/controller/security/firewall`, `/controller/security/access-control`, and
`/controller/security/password-policy` on both `mail.dev.knj.network` and `knj-dnstest-server`.
