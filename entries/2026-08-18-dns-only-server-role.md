# Phase 99 - DNS-Only Server Role

Real cPanel/WHM ships a "DNS Only" build: the same software running in a restricted mode as a
secondary nameserver for a Main server. We wanted the same shape here: one codebase, gated down at
both install and runtime.

## Install-time

The server bootstrap gained a DNS-only role setting. Set it, and the script skips the hosting and
mail stack entirely while still installing the app itself, and installs a smaller DNS setup that
starts empty until linked to a Main server.

## Runtime gating, twice over

The server's role drives enforcement in two independent places. Hiding navigation links alone was
already found insufficient once before on a different feature, so it isn't trusted here either: a
default-deny allowlist of reachable routes, enforced server-side regardless of what the navigation
shows.

## Linking

Adding a DNS-only server on the Main server's Manage Servers page generates a one-time linking
token. Pasting it into the DNS-only server's first-run setup screen completes the link with one
authenticated call, with no manual configuration on either end.

## Keeping zones in sync

Zone content always stays authored on the Main server. Standard DNS replication covers content for
zones the secondary already knows about, but can't discover a brand-new zone, so two mechanisms
work together: a queued, retried job on every zone create and delete, and a periodic reconciliation
sweep as a backstop. A DNS-only server's DNS Replication page is deliberately read-only.

## Verified

The local test suite and code-style checks pass. End to end, a DNS-only server on a disposable test
box was linked to a disposable Main server through the wizard, a hosting account was created on the
Main server, and its zone arrived on the DNS-only server without touching either filesystem by hand.
