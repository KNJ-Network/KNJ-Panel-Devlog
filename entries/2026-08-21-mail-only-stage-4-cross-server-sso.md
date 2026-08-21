# Phase 111 - Mail Only Stage 4: Cross-Server Admin SSO

An admin on Main had no way to reach a linked satellite's own admin area without a second login.
Every satellite, DNS Only or Mail Only, is a separate install with its own database and its own
admin account. The two-key handshake lets Main and a satellite talk for provisioning, but never
gave a person a way to click through.

## Designed to need no extra network trip

The design builds on trust material the linking handshake already established rather than adding
anything new. Main issues a short-lived, single-use sign-in link, and the satellite validates it
independently and consumes it once before signing the admin in, using the same no-password
mechanism account impersonation already uses.

One disclosed, deliberate limitation: the satellite does not know which Main admin initiated the
request and signs in as its own initial admin account. That is fine for these single-tenant
utility boxes and is documented as such.

A "Log in" link appears on Manage Servers next to each linked satellite, and on Mail Settings next
to the active-server selector. Access is initiated from Main only.

## Verified

Full local suite (1669 tests) and the code-style check green. Live: cut v0.16.38, upgraded the
linked Mail Only satellite via its own Panel Updates page, generated a link from Main and opened it
in a browser that had never logged into that satellite; it landed on the real dashboard with no
credentials typed. Reusing the same link returned 403 Forbidden, so single-use protection holds
outside the test suite too.

The feature is role-agnostic: the same code path covers any linked satellite, confirmed by
repeating the sequence against a linked DNS-only test server.

## Stage 3 and Stage 4, both done

This closes out the Mail Only stages: linking, dispatcher, service migration with webmail routing
and Service Status, and now cross-server SSO.
