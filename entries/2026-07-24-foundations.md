# Foundations — 24 July 2026

Starting fresh on KNJ Panel: a hosting control panel built from the ground up as a genuine
alternative to cPanel/WHM, not a reskin of an existing tool. The goal is real feature parity
over time — our own provisioning engine, our own design (inspired by cPanel's look and
information architecture, but our own icons and identity throughout).

## The shape of it

Two layers, matching how WHM/cPanel actually work:

- A **controller** area (the WHM-equivalent) where server admins and resellers manage accounts,
  packages, servers, and security tooling.
- An **account** area (the cPanel-equivalent) that every account — including the primary one —
  gets its own copy of, for managing its own sites, mail, and databases.

Multi-server from day one, matching WHM's server clustering rather than being locked to a single
box. Both areas run on their own dedicated ports (mirroring cPanel/WHM's convention) rather than
sharing the main domain — port 80/443 on the server is reserved for a default landing page, and
eventually for whatever the first hosted account puts there.

## What's actually working today

- A dev server is live, hardened (key-only SSH, firewall, fail2ban, automatic security updates)
  and reachable over HTTPS with a real certificate.
- Accounts, roles (Admin/Reseller/Account), and login are working — no public self-signup,
  accounts are created by an admin/reseller, matching how WHM/cPanel actually work.
- Both panel areas render with a base card-and-sidebar layout, and are properly gated: the
  controller area only exists on its own port and only for Admin/Reseller roles; the account
  area is reachable by any authenticated user, since even the primary account gets its own
  scoped panel.
- The dev server deploys via a real `git pull` from the private source repo now, not manual file
  copying — the actual deployment model this project is meant to use going forward.

## What's next

The core loop: an account gets created, and the system actually provisions a real site for it
(web server config, PHP, a database) end to end. Everything before that is foundation;
everything after it builds on having that loop proven.
