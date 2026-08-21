# Phase 104 - Mail Only, Stage 1: Linking, Role Gate, Bootstrap

## Starting the second satellite role

DNS-only's two-key linking system (Phase 102) and feature-parity pass (Phase 103) proved the
pattern: a restricted-role satellite install links to a Main server, gets pushed what it needs, and
stays gated to only the pages that apply to it. Mail Only is the second satellite role built on
that same foundation — a KNJ Panel install a Main server can hand real mail work off to, rather
than just DNS zone membership.

Mail doesn't get to reuse DNS-only's model unchanged, though. DNS-only works because zone content
is always authored on Main and the satellite only ever receives a push of what already exists —
there's no "create this zone" action a satellite originates. Mail needs genuine remote
*provisioning*: creating mailboxes, generating DKIM keys, writing Sieve scripts. That's real
scope, so the build is staged the same way DNS-only's own work was — link the box for real first,
prove the mechanism, then build out actual mail dispatch on top of a foundation that's already
tested end to end.

## Generalizing instead of duplicating

The two-key handshake itself (`App\Support\ServerLinkKey`) was already role-parameterized — a Link
Key carries which role it's for, and nothing about the crypto cares whether that's `dns_only` or
`mail_only`. What wasn't generalized was everything built *on top* of it: `DnsOnlyLinkService` was
hardcoded to the DNS-only role by name, and `ServerLinkController` 404'd outright for anything
else.

Rather than duplicate a near-identical `MailOnlyLinkService` and a parallel `mail-link` route/page,
both got generalized. `DnsOnlyLinkService` became `ServerLinkService`, taking the expected role as
a parameter instead of assuming it. `ServerLinkController` now resolves the current role from
config and works for either — same route names, same view, same "paste a Link Key, get a Response
Key back" flow, with only the on-page copy (the role's own name) varying. The two DNS-only view
files that lived under `resources/views/whm/dns-only/` moved to `resources/views/whm/satellite/`
to match — one of them (Server Link) now genuinely serves both roles; the other (DNS Replication)
stays DNS-only-specific but sits alongside it for the same reason it always did.

On Main's side, nothing needed to change at all — `ServerController`'s `ADDABLE_ROLES` already
included `MailOnly` from when the two-key scheme was first built, and every action that isn't
DNS-specific (`store`, `completeLink`, `revealLinkKey`, `regenerateLinkKey`, `destroy`) was already
written against the role list, not a hardcoded DNS-only check. Adding a Mail Only server from
Manage Servers, generating its Link Key, and completing the handshake all just worked, unchanged.

## The role gate and what it deliberately excludes

`MailOnlyRoleGate` mirrors `DnsOnlyRoleGate`'s shape exactly — same default-deny route-prefix
allowlist, same Tier-1 server-level pages (Resource Monitor, Service Status, Log Rotation, SSL
Storage Manager, DB Version Upgrade, Security Policies, Diagnostics) already proven safe to expose
to a restricted role. `EnsureAllowedForServerRole` middleware, previously hardcoded to DNS-only
only, now maps `config('panel.role')` to whichever gate class applies — a mechanical, one-line-per-
role change rather than a second copy of the same guard logic.

One thing stays deliberately unexposed: Mail Settings, Mail Relay, Mailserver Config, and the rest
of the mail-configuration nav group aren't reachable on a Mail Only install at all, even though
this box runs the actual mail stack. Same "authored on Main, satellite just runs it" split
DNS-only already established for zone content — a Mail Only box's own local UI stays limited to
server-level status pages, not mail configuration, which will be driven from Main once the
dispatcher (Stage 2) exists.

## bootstrap-server.sh: a third role, not a rejection

`KNJPANEL_SERVER_ROLE=mail_only` was previously hard-rejected with an explicit "not yet supported"
message. That's now a real install path, following the exact shape the `dns_only` branch already
established: self-registers via a new `knjpanel:create-mail-only-server` artisan command (a
near-identical sibling of `create-dns-only-server`, just `role = MailOnly`), skips the account-area
nginx blocks and phpMyAdmin/DNS/FTP stacks entirely, and opens the mail-specific firewall ports
(25/465/587/993) that DNS-only never needs but Main and Mail Only both do.

The one-time mail-stack installer (`setup-mail-server.sh`) needed no changes to be reused here — it
already resolves its own hostname from this server's `is_self` row rather than assuming anything
Main-specific, so a Mail Only box gets Postfix and Dovecot installed and running the same way Main's
own install does, once a hostname and certificate exist (immediately if `DOMAIN` was passed at
bootstrap, or later from Server Setup if not). What it doesn't yet do correctly on a genuinely
separate box is route real mail — its virtual-alias maps query this server's own local
`mailboxes`/`mail_forwarders` tables directly, which are empty on a satellite until something
populates them. That's explicitly Stage 3 scope (a flat generated map file pushed from Main,
replacing the live query), not a Stage 1 gap: this stage is about proving the box exists, links,
and runs the right services — not about mail actually flowing yet.

## Verified

Full local suite (1583 tests) and `pint --test` both green. New coverage: `MailOnlyRoleGateTest`
(mirrors `DnsOnlyRoleGateTest` — allowed routes, Tier-1 exposure, 404s on disallowed routes,
mail-configuration pages specifically confirmed unreachable, nav shape, licence exemption) and new
Mail Only cases added directly into `ServerLinkControllerTest` (a `mail_only`-role link succeeds,
a `dns_only`-role Link Key is correctly rejected when pasted into a `mail_only` install). No live
VM test yet — that's next, once there's something worth linking a real box to see.

Bootstrap-server.sh syntax-checked (`bash -n`); no live install run yet on this stage either, since
there's nothing to provision through it beyond the link itself. Committed and pushed as
`308a3f3`.
