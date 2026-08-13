# Phase 78 - A Feature Toggle That Actually Toggles

Two more items off the gap list: a Perl module installer, and Feature Manager. The first was
straightforward. The second revealed a gap in work that had already shipped.

## Perl modules, the same shape as PHP extensions

The existing PHP extension manager was the obvious template: a curated allowlist, installed state
read directly from the system, and an install action through the provisioning script for the one
step that needs elevated rights. Eight commonly needed modules (database drivers, an HTTP client,
JSON and XML parsing) are validated against an exact allowlist on both ends before anything runs.

## Feature Manager: the page that revealed its own gap

Feature Manager is one consolidated screen listing every hosting package, with a grouped checklist
of every gateable capability. It writes to the same column a package's own edit form uses, so the
two never disagree.

The real work was the checklist itself. Only 6 of roughly 48 account-side capabilities were
gateable, so it was expanded to 26 by adding a feature declaration to 20 previously ungated items.

A follow-up check turned up something worth catching before release: those 20 new toggles only
affected the sidebar. Flipping one off greyed out the link, but the page behind it was still
reachable by URL. The same feature-enabled check the original six used now opens every public
action across all 20 controllers, so a disabled feature returns a 403 rather than just hiding the
link. phpMyAdmin and the core Domains, Email, Databases, and File Manager surfaces stay
deliberately ungated.

New test coverage asserts that routes return 403 for all 20 newly gated features on a package with
the feature disabled, plus a check that an unrestricted package stays fully reachable.
