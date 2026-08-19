# Phase 99 - DNS-Only Server Role

Real cPanel/WHM ships a "DNS Only" build: same WHM software, running in a restricted mode as a
secondary/slave nameserver for a Main server, rather than a separate product. Wanted the same
shape here — one codebase, not a forked repo, gated down at both install and runtime.

## Install-time

`bootstrap-server.sh` gained `KNJPANEL_SERVER_ROLE=dns_only`. Set it, and the script skips the
whole hosting/mail stack entirely — no FTP, no phpMyAdmin, no Roundcube, none of ports
2082/2083/25/465/587/993 — while still installing the app itself. In place of Main's own
authoritative DNS setup it installs a new, smaller `deploy/dns/setup-dns-only-server.sh`: BIND9,
configured secondary-only, empty until linked to a Main server.

## Runtime gating, twice over

`config('panel.role')` drives enforcement in two independent places, deliberately — nav-hiding
alone was already found insufficient once before, on a different feature (see `ServerController`'s
`abort_if($server->is_self, 404)`), so it wasn't trusted alone here either:

- `App\Support\DnsOnlyRoleGate` — a default-deny allowlist of routes a DNS-only install can reach
  at all.
- A dedicated `role.gate` route middleware enforcing that allowlist server-side, independent of
  whatever the nav happens to show (`WhmController::navGroups()` hides the rest, but hiding a link
  was never the actual security boundary).

A DNS-only install is fully exempt from licence enforcement — no accounts, nothing to sell or
meter, does nothing except talk to a Main server that's already separately licensed.

## Linking: one token, one call, done

Adding a DNS-only server in Main's **Manage Servers** page generates a one-time linking token (the
`agent_token` column existed already, from earlier groundwork, but had never actually been used
until now). Pasting that into the DNS-only box's own first-run setup screen makes one authenticated
call back to Main, which hands back the TSIG key, Main's connection info, and the current zone
list — no manual `named.conf` editing on either end.

## Keeping zones in sync

That same token then authenticates Main's own outbound pushes. Two mechanisms, not one, because
BIND's native NOTIFY/AXFR only covers half the problem — it replicates zone *content* once a zone
is already declared `type secondary` on the DNS-only box's config, but has no way to discover a
brand-new zone that box doesn't know about yet:

- `PushZoneMembershipJob` — queued and retried, fired on every zone create/delete on Main.
- `knjpanel:sync-dns-only-servers` — a 15-minute scheduled reconciliation sweep, the backstop for
  anything the immediate push missed.

Zone *content* itself always stays authored on Main. A DNS-only box's own **DNS Replication** page
is deliberately read-only — serials, last transfer time, approximate record count — matching real
usage: it has no local Sites/Accounts to author zones for in the first place.

## Verified

Full local suite and `pint --test` pass. Live end-to-end, for real: added a DNS-only server on
`knj-dnstest-server`, linked it to `knj-test-server` via the wizard, created a hosting account on
the Main box, and watched its zone actually arrive on the DNS-only box without touching either
server's filesystem by hand.

## Why this entry is late

The feature itself shipped 2026-08-18, same day as Phase 98's setup wizard — but only the wizard
got a writeup at the time. This is the one that was actually missing.

## Next

The exhaustive page-by-page audit against real cPanel's DNS Only WHM, to catch any real parity gap
left in the restricted mode itself — not a blocker for anything currently shipped, just unfinished
homework.
