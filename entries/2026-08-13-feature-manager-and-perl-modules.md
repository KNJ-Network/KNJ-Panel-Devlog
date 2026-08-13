# Phase 78 - A Feature Toggle That Actually Toggles

Two more items off the gap list: a Perl module installer, and Feature Manager. The first was
straightforward — the second turned up a real gap in work that had already shipped.

## Perl modules, the same shape as PHP extensions

The existing PHP extension manager was the obvious template: a curated allowlist, installed state
read straight off the live system with no privilege needed (`perl -M{module} -e1`, exit code tells
you everything), and an install action that shells out to the privileged provisioning script for
the one step that actually needs root. Eight commonly needed modules — database drivers, an HTTP
client, JSON/XML parsing — each mapped to its real apt package name, validated against an exact
allowlist on both ends before anything runs.

## Feature Manager: the page that revealed its own gap

Feature Manager itself is simple: one consolidated screen listing every hosting package down the
side, a grouped checklist of every gateable capability, save. It writes to the exact same column a
single package's own edit form already used, so the two never disagree.

The real work was in the checklist itself. Only 6 of roughly 48 account-side capabilities were
gateable at all — everything built since then had simply never been wired into the toggle. Getting
Feature Manager to be honestly useful meant expanding that to 26, adding a `'feature' => 'key'`
declaration to 20 previously-ungated items: Directory Privacy, Deploy from Git, Domain Forwarding,
Dynamic DNS, Quick Site, Spam Scoring, Greylisting, Mailing Lists, Mail Encryption,
Challenge-Response, the Database Browser, Traffic & Visitors, IP Blocker, Hotlink and Leech
Protection, the MultiPHP INI Editor, Optimize Website, Indexes, Error Pages, and MIME Types.

Wiring up the checklist wasn't the whole job, though. A quick check afterward —
`grep -rln "ensureFeatureEnabled" app/Http/Controllers/Account/` — turned up something worth
catching before it shipped: every one of those 20 new toggles only ever affected the sidebar. Flip
one off and its nav link greyed out correctly, but the page behind it was still fully reachable by
anyone who already had the URL or a bookmark. The toggle looked real and wasn't. Fixed by adding the
same `ensureFeatureEnabled()` call every one of the original 6 gated controllers already opened
with, as the literal first line of every public action across all 20 controllers, so a disabled
feature now actually 403s the request rather than just hiding the link to it. phpMyAdmin and the
core Domains/Email/Databases/File Manager surfaces stay deliberately ungated, unchanged — a package
was never meant to be able to hide those.

New test coverage went in specifically to catch a regression of exactly that gap: a
`FeatureGateEnforcementTest` builds an account on a package with one feature disabled and asserts
the route 403s, for all 20 newly-gated routes at once, plus a spot-check that a package with no
restrictions (`features: null`) stays fully reachable.

## Next

Continuing down the gap list.
