# Phase 56 - Server Configuration, and What "Done" Actually Means Here

Five items on this section's roadmap. Two of them were already fully built and just mislabeled.
One was a genuine, straightforward build. The last two turned out to be the wrong shape of task
entirely — worth explaining why, since the answer changes how the rest of this roadmap gets closed
out.

Server-wide settings and Timezone configuration were both already live — the Server Settings page
has covered hostname, timezone (via `timedatectl`), upstream DNS resolvers, contact/notification
emails, and a graceful reboot button since late July. Neither had ever been marked correctly on the
public roadmap. Same category of miss as a few sections back: found by actually reading the code
before building anything, not by trusting the roadmap's own status pills.

Traffic & visitor statistics was the one real build — a new page per account, linked from its row
on List Accounts, reporting requests, unique visitors, bandwidth, and a response-status breakdown
for a chosen month, plus top pages and top referrers. Built on the exact same Nginx access-log
parsing the existing bandwidth tracker already does — same file-finding logic, same stateless
re-sum-the-whole-month approach, just pulling request/visitor/referrer detail out of each line
instead of only summing bytes. No new privilege needed, since those logs were already
app-readable. Live-verified with real traffic: created a disposable test account, fired real curl
requests at its domain (including one with a deliberately missing path and one with a fake
referrer), and confirmed the numbers on the page matched exactly — down to catching and fixing an
actual capture-group indexing bug in the parser before it ever shipped, and a red herring where the
first verification attempt went out over HTTPS to a domain that had no TLS vhost yet, silently
landing on a different site's log entirely.

The last two — Server role profiles and Multi-server support — are where this section stopped being
a normal roadmap checklist. Both already have real, shipped foundations: a `role` field
(Main/DNS-only/Mail-only) on every registered server, full CRUD for adding and removing them, and —
for DNS-only specifically — a real DNS clustering feature that generates TSIG keys and pushes BIND
primary/secondary replication config automatically the moment a DNS-only server is registered. What
neither one actually has is *role-aware provisioning* — a server genuinely configured differently
depending on what it's for, rather than just labeled. That turns out to be exactly the same work as
the dedicated DNS-only server build already planned as its own separate project: a stripped-down
fork of this repo, its own image, its own VM, tested by actually linking it back to a real main
server. Building a shallow version of that inside this section would either not do anything real,
or just be the DNS-only project itself done early and out of order. Marked both honestly on the
roadmap — foundation shipped, completion waits on that project — rather than force a checkbox that
wouldn't mean anything.

That's the admin-side roadmap done, as far as this pass goes. Client Panel is next.
