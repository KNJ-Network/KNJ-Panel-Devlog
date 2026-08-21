# Phase 104 - Mail Only, Stage 1: Linking, Role Gate, Bootstrap

## Starting the second satellite role

DNS-only's two-key linking system (Phase 102) and feature-parity pass (Phase 103) proved the
pattern: a restricted-role satellite install links to a Main server, gets pushed what it needs, and
stays gated to only the pages that apply to it. Mail Only is the second satellite role built on
that foundation, a KNJ Panel install a Main server can hand real mail work off to.

Mail cannot reuse DNS-only's model unchanged. DNS content is always authored on Main and pushed;
mail needs genuine remote provisioning: creating mailboxes, generating DKIM keys, writing Sieve
scripts. That is real scope, so the build is staged: link the box for real first, prove the
mechanism, then build mail dispatch on top.

## Generalizing instead of duplicating

The two-key handshake was already role-parameterized. What was not generalized was everything built
on top of it: the link service and page assumed the DNS-only role. Both now take the role as a
parameter and work for either, with only the on-page copy varying. On Main's side nothing needed to
change; adding a Mail Only server, generating its Link Key and completing the handshake worked
unchanged.

## The role gate and what it deliberately excludes

The Mail Only role gate mirrors the DNS-only one: default-deny, with the same server-level pages
already proven safe to expose to a restricted role. Mail configuration pages are deliberately not
reachable on a Mail Only install, even though the box runs the mail stack: configuration is
authored on Main and the satellite just runs it.

## The install script: a third role, not a rejection

Installing as a Mail Only server was previously rejected with a "not yet supported" message. It is
now a real install path following the DNS-only branch's shape: it self-registers, skips the
account-area web server blocks and the DNS, FTP and database-admin stacks, and opens the mail
firewall ports the role needs. What it does not yet do correctly on a separate box is route real
mail, since lookups depend on data a satellite does not have until Main pushes it. That is Stage 3
scope; this stage proves the box exists, links, and runs the right services.

## Verified

Full local suite (1583 tests) and the code-style check green, with new tests for the Mail Only
role gate and for Link Key role mismatches being rejected. The install script was syntax-checked;
no live install yet at this stage. Committed and pushed.
