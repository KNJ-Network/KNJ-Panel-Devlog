# Phase 103 - Closing Out DNS-Only Feature Parity

## Starting from a real gap

The two-key linking system (Phase 102) fixed how a DNS-only box connects to a Main server, but
left an obvious hole: once linked, there was no way back into that screen if the link ever needed
redoing — a box pointed at the wrong Main server, or reinstalled, had nowhere to go. That was the
starting point for a wider pass: go back through the DNS-only role and close out what was still
genuinely missing, checked directly against a real cPanel DNSOnly install rather than assumed.

## One page instead of two

DNS-only server linking used to be two separate screens: a first-run setup page that generated a
Link Key and redirected away the moment linking completed, and a second, permanently read-only
status page for afterward. The setup screen becoming unreachable once linked was the actual bug —
the underlying linking service already supported re-linking cleanly (`updateOrCreate` under the
hood), the gap was purely that the UI never gave you a way back to it.

Both screens are now one: `DnsReplicationController` renders a single "DNS Replication" page that
shows the zone status table plus a collapsible re-link form once linked, or the paste-a-Link-Key
form directly when not. Same page, no dead ends.

## Hostname drift, fixed in both directions

Two related bugs got caught while working through this. First: a DNS-only box's own hostname
never updated after its first bootstrap, because the sync code that keeps a self-row's hostname
current filtered by `role = main` — a role check that had no business being there, since exactly
one row per install ever has `is_self = true` regardless of which role that install is. Second:
once a DNS-only box's hostname *did* change (a real domain configured after an initial IP-only
install), Main never found out short of a full re-link. Both now flow through the existing
15-minute sync heartbeat instead — the satellite reports its current hostname on every
authenticated push, and Main self-corrects its stored copy if it's changed, the same pattern
already used for delivering the TSIG key on every sync rather than just the first.

## Seven features that were already built

Checking DNS-only against real cPanel DNSOnly's own feature set surfaced seven admin pages already
built for Main that had no real reason to be Main-only: Resource Monitor, Service Status, Log
Rotation, SSL Storage Manager, Upgrade Database Version, Security Policies, and Diagnostics. All
seven are genuinely server-level — none of them touch an Account, a Site, or a hosting database,
the actual boundary a DNS-only install draws. Confirmed directly against what `bootstrap-server.sh`
installs on this role (nginx, MariaDB, PHP-FPM, BIND9, fail2ban, SSH — the panel's own stack, needed
regardless of role) before wiring them in, rather than assuming.

That check also caught a real bug: Service Status showed postfix, Dovecot, and vsftpd as
permanently "not set up yet" on a DNS-only box, because the code that decides "installed but
stopped" vs. "never installed" only understood one reason for something being absent — no domain
configured yet, Main's own story. On DNS-only those three are never installed at all, domain or
not. Fixed by dropping them from the list entirely on this role, rather than showing a state that
can never resolve itself.

## A manual sync button

The 15-minute `knjpanel:sync-dns-only-servers` sweep is the real safety net, but waiting up to 15
minutes to confirm a freshly-added zone or a hostname fix actually landed is genuinely annoying
during normal admin work. Main's own Manage Servers edit page now has a "Sync now" button next to
Test Connection for any linked DNS-only server, which just dispatches the existing
`PushZoneMembershipJob` on demand instead of waiting for the next scheduled run.

## One planned item, dropped after checking it against reality

The build order also included a self-heal "Add an A Entry for hostname" helper, modeled on a real
WHM feature. Checking it against this codebase first turned up two problems: DNS-only boxes can't
author DNS zones at all (only Main can — zone authoring is blocked entirely by the DNS-only role
gate), and a panel's own hostname typically isn't backed by a zone this install controls in the
first place, since real-world DNS for that hostname usually lives on infrastructure entirely
outside the panel. Building the feature as scoped would have meant UI for a state that essentially
never applies. Dropped rather than shipped for the sake of shipping something.

## Verified for real, not just tested

Every piece here went out as a real release (v0.16.22, v0.16.23) and was installed on
`knj-dnstest-server` via its own `knjpanel-upgrade` before being called done — the merged
Replication page confirmed showing its real linked state (status card, zone table, collapsible
re-link form), the sidebar's DNS Only pill confirmed stacking correctly above the version pill, and
all seven newly-exposed pages confirmed rendering cleanly with the trimmed six-service list, not
just passing against a database that had never seen the old scheme or the old service list in the
first place.
