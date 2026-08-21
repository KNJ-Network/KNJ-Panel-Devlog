# Phase 111 - Mail Only Stage 4: Cross-Server Admin SSO

The last open item from the Mail Only roadmap: an admin on Main had no way to reach a linked
satellite's own WHM-equivalent without a second login. Every satellite — DNS Only or Mail Only — is a
fully separate Laravel install with its own database and its own admin account. The two-key mutual-
auth handshake already built for linking (`ServerLinkKey`) gets Main and a satellite talking to each
other for provisioning dispatches, but never gave a human a way to just click through.

## Signing entirely offline

The design leans on the same trust material the linking handshake already established, rather than
inventing anything new. Main's row for a satellite holds `local_secret` — the value Main presents to
authenticate itself when calling out. The satellite's row for Main holds only
`hash('sha256', local_secret)`, never the raw value. That asymmetry is deliberate (it's the same shape
`MailDispatchController` already verifies against), and it shapes the whole design: a new
`ServerSsoService::mintUrl()` signs a short-lived (60-second), single-use token entirely locally on
Main — no network round trip — keyed on that same hash, since that's the only material both sides can
independently reproduce. The satellite's new `SsoController::redeem()` recomputes the identical HMAC
from its own stored hash, checks the server-computed expiry, and consumes the token atomically via
`Cache::add()` before calling `Auth::login()` — the exact no-password primitive account impersonation
already uses, just triggered by token validation instead of a button click.

One disclosed, deliberate limitation: the satellite has no idea *which* Main admin initiated the
request. Main is a trusted peer vouching for an authorized admin, so the satellite signs in as its own
oldest (first-created) admin account. Fine for these single-tenant utility boxes — the setup wizard
only ever creates one initial admin — and documented as such rather than pretended away.

A new "Log in →" link shows up on Manage Servers next to each linked satellite, and on Mail Settings
next to the active-server selector. Main-initiated-only, matching every other piece of this linking
model.

## Verified

Full local suite (1669 tests) and `pint --test` green. Live, for real: cut v0.16.38, upgraded
`mail.dev.knj.network` via its own Panel Updates page, then minted a real SSO URL from Main and opened
it in a browser session that had never logged into that satellite — landed straight on its real
dashboard (`knj-mail-test-server`, 2/2 servers healthy, its own Service Status grid), fully
authenticated, no credentials typed anywhere. Redeeming the exact same URL a second time returned a
real 403 Forbidden — single-use replay protection holds outside the test suite too, not just inside
it.

This is genuinely role-agnostic, not a mail-only feature that happens to be named after the mail-only
stage it shipped alongside — the same code path covers any linked satellite. Confirmed by also
upgrading the linked `knj-dnstest-server` (DNS Only, a big jump straight from v0.16.24 to v0.16.38
since that box hadn't been touched since Stage 1) and repeating the exact same sequence: minted URL,
fresh redemption landed on its real DNS Only dashboard (DNS Replication/Server Link nav visible,
confirming it's genuinely that role's own UI and not a generic fallback), then the second redemption
attempt also correctly came back 403.

## Stage 3 and Stage 4, both done

This closes out everything tracked under Mail Only in this session: Stage 1 (linking), Stage 2
(dispatcher), Stage 3 (every remaining mail service migrated, webmail routing fixed, Service Status
split), and now Stage 4 (cross-server SSO). What's left isn't scoped to any of these stages: plain
forwarders' own auth-map push, DKIM, and mail statistics all remain open, tracked separately.
