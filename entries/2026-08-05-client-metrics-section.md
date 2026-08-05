# Phase 63 - The Client Metrics Section, Closed Out

Five roadmap rows, three of which turned into two features, plus one that just needed the roadmap
told the truth.

Raw access log download went first, and took thirty seconds — it had shipped back on 2026-07-29 as
part of Raw Logs, and the roadmap row was simply never flipped when it did. Same pattern that's
shown up a few times this session now: build the feature, forget the paperwork.

The error log viewer was the one that actually needed building. `Account\LogDownloadController`
already let an owner download their domain's full error log, but "download the whole file" and
"see what just broke" are different jobs — nobody wants to pull a multi-megabyte file to check if
today's deploy is throwing errors. Confirmed first, rather than assumed, that Nginx's log files are
world-readable (0644) on the real server, which is exactly why `VisitorStatsService` and
`UsageService` already read them directly with no `sudo` involved — so the new viewer does the same
thing: read the current file in-process, no new provisioning-script action needed at all. Shows the
last 200 lines, newest first, with a site selector for accounts running more than one domain.

Bandwidth usage already had an account-wide total on the Disk & Bandwidth page from a previous
phase; the only real gap was a per-domain breakdown. The logic already existed too — buried as a
private method inside `UsageService` that summed every domain's bytes into one number. Pulling it
out into a public `bandwidthByDomain()` and letting the controller keep each domain's total
separately, rather than summing them, was the entire change.

The last two rows — Visitor summary and Traffic statistics dashboard — turned out to be the same
feature wearing two names. Both draw from `VisitorStatsService::statsForDomain()`, which was
already built and already working on the Controller side for admins looking at one account's
traffic. Building two near-identical account-side pages for two roadmap rows that ask for the same
data would've been pure roadmap-satisfying busywork, so they became one: a Traffic & Visitors page
with a domain + month selector, requests/visitors/bandwidth/error-response totals, a response-status
breakdown, and top pages/referrers tables — mirroring the admin view's layout closely enough that
anyone who's used one has already used the other.

All three new pages went through the same live-verification loop as everything else this session:
a real disposable account created through Controller, "log in as" into the account side, confirm
the page renders against the real deployed code (not just the test suite), then torn back down.
Nothing dramatic to report from that pass — no 500s, no missing views, correct empty states when a
brand-new domain has no traffic or errors yet.

Metrics section closed out. Five for five, two of them merged into one honest page instead of
staying two separate roadmap rows pretending to be different features.
