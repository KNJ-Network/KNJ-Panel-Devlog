# Phase 01 - Foundations

Starting fresh on KNJ Panel: a hosting control panel built from the ground up. The goal is a
genuinely complete feature set over time — our own provisioning engine, our own design and
identity throughout.

## The shape of it

Two layers:

- A **Controller** area where server admins and resellers manage accounts, packages, servers, and
  security tooling.
- An **account** area that every account — including the primary one — gets its own copy of, for
  managing its own sites, mail, and databases.

Both areas are served separately from the hosted sites' default landing page.

## What's actually working today

- A dev server is live, hardened, and reachable over HTTPS with a real certificate.
- Accounts, roles (Admin/Reseller/Account), and login are working — no public self-signup,
  accounts are created by an admin/reseller.
- Both panel areas render with a base card-and-sidebar layout, and are properly gated by role.
- The dev server deploys via a real `git pull`, not manual file copying — the deployment model this
  project is meant to use going forward.

## What's next

The core loop: an account gets created, and the system actually provisions a real site for it
(web server config, PHP, a database) end to end.
