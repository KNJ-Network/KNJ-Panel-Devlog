# Phase 161 - The Address Bar Problem

Reported directly by the operator, testing their own real account: visit `knj.network/webmail`,
and the browser's address bar changed to `https://ws1.knj.network:2083/webmail/1?folder=INBOX`.
Login worked fine. That was the problem. A white-label multi-tenant host has no business putting
its own hostname in a customer's browser bar, ever, and `/webmail`, `/account`, `/controller`, and
the `webmail.`/`account.` subdomain shortcuts had all been doing exactly that since the day they
shipped.

## Why a redirect was ever the right call, until it wasn't

The original reasoning (still sitting in `NginxSettingsService`'s own doc comment until today) was
that a redirect wasn't a shortcut around a hard problem — the target already had valid HTTPS on the
panel's own hostname, so there was no TLS/SNI mismatch or cookie-domain mismatch to work around, and
a reverse proxy would have introduced both. All true. It just optimized for the wrong thing: making
the plumbing simple instead of keeping the customer's own domain in their own address bar, which is
the entire point of white-labeling. Once that was named out loud, simple wasn't good enough anymore.

## Building the proxy without breaking the two things a redirect gets for free

A redirect needs no cookie-domain story and no real-IP story, because the browser does a fresh
top-level navigation to a domain that already has both sorted. A proxy inherits neither for free.

Cookie domain first: `config/session.php`'s `SESSION_DOMAIN` is `null`, meaning the session cookie
scopes to whatever `Host` header the login request arrived with. A proxy that doesn't forward `Host`
unchanged would silently set the cookie for the *panel's* hostname while the browser thinks it's
talking to the customer's domain — the cookie would exist, just never come back on the next request.
Fixed by forwarding `Host` explicitly in every `proxy_pass` block.

Real IP second, and this one's less obvious until you trace where it's used: fail2ban, login-lockout,
and Security Questions' known-IP recording all read `$request->ip()`. Without `X-Real-IP`/
`X-Forwarded-For` forwarded *and* Laravel told to trust the hop that's forwarding them, every one of
those would have started logging every proxied login as coming from `127.0.0.1`. Added both —
`proxy_set_header X-Real-IP $remote_addr;` (plus `X-Forwarded-For`/`X-Forwarded-Proto`) on the nginx
side, and `$middleware->trustProxies(at: ['127.0.0.1', '::1'])` in `bootstrap/app.php` on the Laravel
side. `::1` is there pre-emptively — the actual `proxy_pass` target today is IPv4-literal, but a
future IPv6 loopback hop should never silently stop being trusted just because nobody remembered to
add it.

One more wrinkle: the loopback leg's own TLS. `proxy_pass https://127.0.0.1:2083/webmail` connects
to a certificate issued for the panel's own hostname, not for `127.0.0.1` — verifying it against the
connection address would always fail. `proxy_ssl_verify off` on that one hop is safe specifically
*because* it's loopback, the same reasoning `DirectoryPrivacyService`'s Leech Protection auth
subrequest already relies on elsewhere in this codebase. This is just the first time that pattern
carries a full interactive, cookie-bearing, CSRF-protected session instead of an `internal;`
auth-only check.

## webmail./account. need a real certificate; controller. never should

Proxying the *paths* (`/webmail`, `/account`, `/controller`) needed none of that SAN complexity —
they ride the customer's already-valid certificate for their own domain. The subdomain shortcuts
(`webmail.<domain>`, `account.<domain>`) are a different animal: a customer visiting
`https://webmail.knj.network` needs a certificate that actually lists that name, or they hit a
browser warning before ever reaching the proxy.

So `AccountProvisioningService::issueSsl()` now requests `webmail.$domain` and `account.$domain` as
extra SANs, and `knjpanel-provision-account`'s `ssl` action degrades through three tiers instead of
two — full set, then apex+www only, then apex only — since certbot's multi-name issuance is
all-or-nothing and one slow-to-propagate DNS record shouldn't cost the account its certificate
entirely. `controller.$domain` never gets this. That's not an oversight, it's the one shortcut this
change deliberately leaves alone: putting the admin/WHM entry point's hostname on a
customer-controlled certificate is exactly the boundary this codebase keeps avoiding everywhere
else, and real cPanel draws the same line — AutoSSL leaves `whm.<hostname>` off every account
certificate by default, for the same reason. `controller.<domain>` stays a plain redirect. The
`/controller` *path* still proxies, same as `/webmail` and `/account` — it's only the subdomain
shortcut that's carved out.

Subdomain sites are carved out entirely, both directions: `Site::isSubdomain()` sites have no DNS
zone of their own, so a `webmail.`/`account.` SAN request for one could never pass an HTTP-01
challenge — and since certbot's SAN request is all-or-nothing, that would fail issuance for the
*whole* certificate, not just quietly skip the extra name.

## The backlog nobody would have noticed

Every new account gets these vhost blocks and SANs at creation time now. Nothing was ever going to
go back and fix the accounts that already existed — including live production ones. Added
`ExpandServiceSubdomainCertificates`, a scheduled sweep (`everyFifteenMinutes`) scoped to
Let's-Encrypt-sourced, Active, non-subdomain sites: reads each one's on-disk certificate SANs with a
plain `openssl x509` call (cheap, local, no ACME rate limit to worry about), skips anything that
already has both names, and for anything that doesn't, prepares the vhost then dispatches a reissue.
Capped at 20 dispatches per tick — not 20 in the query itself, which would get stuck re-checking the
same static first 20 forever once those happened to already be fixed. Capping the dispatch count
instead means every tick genuinely advances the backlog, clearing it in a handful of ticks with zero
manual action, on an install with any number of eligible sites.

## Verifying it twice — once safe, once for real

panel-dev first: a full CSRF-protected login flow run against a disposable mailbox on
`test.knj.network`, confirmed by `Location:` headers and cookie scoping that the address bar stayed
on `test.knj.network` the entire way through. A deliberate failed login confirmed `TrustProxies` was
actually wired correctly — found the real client IP, not `127.0.0.1`, sitting in `panel_auth.log`.
The `webmail.`/`account.` subdomain proxy blocks worked correctly over plain HTTP; the SAN step
degraded through the tiered fallback exactly as designed when Let's Encrypt returned NXDOMAIN for
those names on that particular dev-only domain's DNS — a fact about that test domain, not a bug.
`controller.` was confirmed unchanged, still a plain 301.

Then the real thing. Released as v0.17.12, deployed to all four production servers through Panel
Updates — Main via Update Now, then the bulk "Update all linked servers" button (shipped last phase)
for the three satellites, all of which succeeded this time since every one of them was already on
v0.17.11+, the version that carries the receiving endpoint that button needs. Live-verified against
the actual `knj.network` account: created a disposable `webtest@knj.network` mailbox, ran the same
full CSRF-protected login flow, and confirmed it stayed on `knj.network` throughout — previously it
redirected to `ws1.knj.network:2083` the whole way. Confirmed the real homepage still returned 200
after the nginx snippet regenerated, then deleted the disposable mailbox to leave the account clean.

One nuance worth writing down before it's forgotten: the nginx snippet that carries this fix is only
regenerated when Nginx Settings is explicitly re-saved (WHM → Server Configuration → Nginx Settings
→ Save & Apply). Upgrading the panel's own version doesn't retroactively rewrite already-generated
vhost config sitting on disk — that Save & Apply click was a required one-time step on both
panel-dev and `ws1` after deploying the code that changed what `buildSnippet()` writes. Not a defect,
just a step that's easy to forget exists.

Tested (2543/2543).
