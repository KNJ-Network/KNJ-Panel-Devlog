# Phase 103 - Closing Out DNS-Only Feature Parity

## Starting from a real gap

The two-key linking system (Phase 102) fixed how a DNS-only box connects to a Main server, but
left an obvious hole: once linked, there was no way back into that screen if the link ever needed
redoing. That was the starting point for a wider pass: go back through the DNS-only role and close
out what was still genuinely missing, checked against the feature set of a comparable DNS-only
product rather than assumed.

## From two pages, to one, to two again

DNS-only server linking used to be two separate screens: a first-run setup page and a permanently
read-only status page for afterward. The setup screen becoming unreachable once linked was the
actual bug. The first fix merged both into one page, but the combined page read as "resync zones"
rather than "manage the link." Splitting it back into two purpose-named pages fixed that:
**Server Link** (under the sidebar's Server group) owns the linking status and the re-link form;
**DNS Replication** (its own DNS group) owns the read-only zone table. Both show the same
"linked to X" summary.

## Hostname drift, fixed in both directions

Two related bugs turned up. A DNS-only box's own hostname never updated after its first
bootstrap, and once it did change, Main never found out short of a full re-link. Both are now
handled by the regular sync heartbeat, with Main correcting its stored copy when the satellite
reports a new hostname.

## Seven features that were already built

Comparing against a real DNS-only product surfaced seven admin pages already built for Main that
had no reason to be Main-only: Resource Monitor, Service Status, Log Rotation, SSL Storage
Manager, Upgrade Database Version, Security Policies, and Diagnostics. All are genuinely
server-level. That check also caught a real bug: Service Status showed mail and FTP services as
permanently "not set up yet" on a DNS-only box, where they are never installed at all. They are
now left out of the list on that role.

## A manual sync button, reachable from both sides

Waiting for the periodic sync to confirm a freshly added zone is annoying during admin work.
Main's Manage Servers edit page got a "Sync now" button for linked DNS-only servers, and the
DNS-only side got a matching button that asks Main to push immediately, using the same two-key
trust relationship the linking system already runs on.

## One planned item, dropped after checking it

A self-heal "Add an A Entry for hostname" helper turned out not to apply: DNS-only boxes cannot
author zones, and a panel's own hostname usually lives in DNS outside the panel. We dropped it
rather than ship a feature for a state that essentially never applies.

## Verified for real, not just tested

Every piece went out as a real release (v0.16.22 through v0.16.24) and was installed on a
DNS-only test server via its own upgrade script. Live testing of the new Sync Now button caught a
bug the automated tests missed: the new internal endpoint was rejected on every real request
because it lacked an exemption its sibling internal endpoints already had. Fixed, redeployed, and
reconfirmed between two real boxes.
