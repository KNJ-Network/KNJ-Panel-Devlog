# Phase 117 - DNS-only comparison audit

Last item on the day's list: a page-by-page comparison of a standard DNS-only nameserver product
against our own DNS-only role, to catch anything a genuine nameserver operator would expect that we
had left out. It worked from the product's public documentation, cross-checked against our role
allowlist and the real controllers behind each candidate route, since a route existing does not
mean it is relevant to a satellite with no hosting accounts.

## What was already covered

Most of the documented feature set was already reachable from earlier parity work: hostname,
resolver and contact management; reboot and per-service restarts; DNS cluster configuration and
sync (our two-key architecture does the same job as the other product's trust relationship); log
rotation, SSL Storage Manager, Security Policies, Diagnostics and updates; and Two-Factor
Authentication.

Correctly out of scope by design: a support-ticket system, a multi-IP management page, and a DNS
software selector.

## The real gap

**Firewall, Access Control, and Password Policy were missing from both satellite roles'
allowlists.** All three are role-agnostic security pages already working on Main that apply just as
directly to a satellite's own SSH and panel logins:

- **Firewall**: brute-force-protection status and unban. Without it, an admin whose satellite got
  a legitimate IP blocked had no way to see or clear that ban except SSH, against this panel's goal
  of not requiring operator SSH for day-to-day management.
- **Access Control**: per-service IP allow/deny, most useful for the services each satellite role
  exposes.
- **Password Policy**: the requirements for the box's own admin login.

All three were added to both roles' allowlists with matching nav entries, since an allowlist entry
alone would have left the pages reachable by URL but invisible in the sidebar.

## Confirmed not gaps

- **API Token management**: every ability is a Main-only hosting concept with no equivalent on a
  satellite.
- **DNS zone TTL/SOA defaults**: these apply to new zones, and satellites never originate zones.

## Flagged, not fixed

Service Certificates is a legitimate gap, but its list of shared services is not role-aware and
would mislead on a DNS-only box. It needs a role-aware list first.

## Verified

Full local suite (1,703 tests) and the code-style check green, with dedicated coverage for both
roles' gates. Deployed as v0.16.44 to the test servers and checked for real against both linked
satellites: six checks, all successful.
