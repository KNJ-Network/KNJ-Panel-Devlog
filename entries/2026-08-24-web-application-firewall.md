# Phase 128 - Web Application Firewall (ModSecurity + OWASP CRS)

A real WAF, not a lightweight pattern-matching substitute: ModSecurity v3 as an nginx dynamic module,
running the OWASP Core Rule Set. The target distribution ships no prebuilt package for either piece,
so this is a from-source build, while the panel-side surface stays small: a per-site mode selector
(Enabled, Detection Only, Disabled, Inherit), one server-wide settings page, and a blocked-requests
log view on both the account and admin sides.

## What shipped

A server-wide default with a per-site override, since opt-in-only would mean most accounts never turn
it on and no override would make a false positive an admin-only fix. The engine install is a long
queued job that builds pinned versions and pins the web server package afterward so unattended
upgrades cannot silently break module compatibility. A scheduled health check verifies the module is
present, loaded and inspecting traffic, because configuration-only checks cannot catch "loaded but
silently not inspecting."

Blocked requests are read live from the WAF's own audit log, filtered by domain on the server side
before anything reaches PHP. That log is shared across all domains and its entries can include
request bodies, so broader read access would leak cross-customer data. There is no database ingest
pipeline, matching the panel's "query live state, don't duplicate it" precedent.

## Two real from-source build bugs, caught live

Neither showed up until an actual install ran on the dev server. The first was a build dependency
mismatch where the library's configure step needed an older regular-expression library than the one
installed. The second was that reusing the web server's own build arguments, the documented approach
for ABI-matched modules, also re-validates unrelated modules and failed on a missing development
library.

## A bug that made the WAF a no-op

During end-to-end verification on the dev server, attack test payloads against a site set to Enabled
returned success instead of being blocked, even though the engine looked healthy. The cause was that
the internal health-check configuration loaded the same ruleset a second time, which made the web
server's config test fail on every regeneration; the install job logged the failure but still showed
as succeeded. The health-check configuration now inherits the single ruleset like every real site.

## Verified

Live on the dev server against a disposable test domain: test attack payloads were blocked once the
fix was deployed and the homepage kept working; Detection Only mode logged matches without blocking
and flipping back to Enabled blocked immediately; the blocked-requests view parsed real entries; and
the fail-closed behavior held when the module file was removed (the config test failed as designed
and traffic was unaffected) and recovered when it was restored.

23 new tests, the suite green and code-style clean. Shipped across several commits as each build issue
was found and fixed live.
