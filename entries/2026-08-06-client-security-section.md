# Phase 64 - The Client Security Section, Closed Out

Two roadmap rows, one real gap, and a genuine "only the primary domain" bug hiding underneath both
of them.

The roadmap read "SSL/TLS: certificates are live, CSR generation is next" and "SSL/TLS status
overview: single-domain view is live, bulk view is next." Reading the actual code first, both notes
turned out to be half wrong. CSR generation already existed — it shipped back on 2026-07-30, before
this phase even started, and the admin-side CSR tool built earlier this session explicitly reused
it rather than writing a second implementation. So "CSR generation is next" was stale the moment it
was written.

What was genuinely missing sat one layer deeper: `Account\SslController` only ever looked at the
account's `primarySite`. An addon domain — a real, supported thing an account can have several of —
had zero certificate visibility or control. Not "the UI doesn't show it," but the controller itself
never queried for it. Both roadmap rows were really the same underlying bug wearing two different
descriptions: "per-domain certificates" was missing because there was no per-domain anything, and
"bulk status overview" was missing for the identical reason.

Fixed as one change. Every mutating action — retry, CSR generation, custom certificate install — now
resolves its site from a posted `site_id`, checked against the account's own sites before anything
runs, so one account can't reach another's certificate by guessing an ID. The page gained a bulk
table across every domain (status, issuer, expiry) with a "Manage" link per row that switches the
detail section below to that domain — cPanel's own SSL/TLS Status page works the same way, list then
click to manage, so this wasn't a novel pattern to invent.

Live-verified with a disposable account that actually had two domains — a primary plus a real addon
domain added through the account's own Domains page — rather than trusting the single-site test
coverage to stand in for it. The bulk table showed both, "Manage" on the addon domain correctly
swapped the detail section over, and CSR generation ran for real against the addon domain and
returned a genuine CSR and private key. That's the exact path that was structurally impossible
before this phase.

Security section closed out. Everything else in it — IP blocking, hotlink protection, directory
login protection, account-level API tokens — had already shipped in earlier phases.
