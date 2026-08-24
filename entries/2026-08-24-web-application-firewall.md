# Phase 128 - Web Application Firewall (ModSecurity + OWASP CRS)

A real WAF, not a lightweight nginx-regex substitute: ModSecurity v3 compiled as an nginx dynamic
module, running the actual OWASP Core Rule Set. Ubuntu 24.04 ships no prebuilt package for either
piece, so this is a from-source build — real work, but the panel-side surface stays small: one
per-site mode selector (Enabled / Detection Only / Disabled / Inherit), one server-wide settings
page, and a blocked-requests log view on both the account and admin sides.

## What shipped

Server-wide default posture (on by default, paranoia level 1) with a required per-site override —
opt-in-only would mean most accounts never turn it on; no escape hatch would make a false positive
an admin-only fix. The engine install itself is a real 15-40+ minute queued job (`InstallWafEngineJob`)
that clones pinned tags of libmodsecurity3, the ModSecurity-nginx connector, and OWASP CRS, builds
the connector against the exact captured `nginx -V` configure arguments (nginx.org's own documented
practice for ABI-matched dynamic modules), and pins the nginx apt package afterward so
`unattended-upgrades` can never silently break the module's ABI compatibility. A scheduled health
check (`knjpanel:check-waf-health`, every five minutes) does three real checks in order — module
file present, module actually loaded, ruleset actually included — then sends a real synthetic
SQL-injection request against a loopback-only diagnostic vhost and confirms it gets blocked, since
none of the config-only checks can catch "loaded but silently not inspecting traffic."

Blocked-requests are read live from ModSecurity's own audit log, server-side, filtered by domain
before anything reaches PHP — the audit log is one shared root-owned file across every domain on the
box, and CRS's `RelevantOnly` entries include full request bodies, so granting the panel's own
file-group read access the way per-domain nginx error logs already work would leak cross-customer
data. No DB-ingest pipeline either, matching this panel's existing "query live state, don't
duplicate it" precedent for fail2ban/Firewall.

## Two real from-source build bugs, caught live

Neither showed up until an actual install ran on `panel-dev` — both are exactly the kind of thing
that only exists once you're really compiling against a real distro's real package set.

**libmodsecurity wants PCRE1, not PCRE2.** The build failed with "pcre library is required" despite
`libpcre2-dev` being installed — libmodsecurity's own `./configure` specifically probes for the
classic `libpcre3-dev` (pcre.h/libpcre.so), a different library entirely from PCRE2, and gives a
misleading error when only the newer one is present.

**Reusing nginx's own configure arguments re-validates modules that have nothing to do with
ModSecurity.** The documented way to build an ABI-matched dynamic module is to reuse the target
nginx binary's own captured `nginx -V` configure-arguments string verbatim. The catch: Ubuntu's
stock nginx package is built with several *other* `--with-*=dynamic` modules too
(geoip/image_filter/perl/xslt/mail/stream), and reusing the full argument string means nginx's own
`./configure` re-validates every one of them — not just the one being added. Without
`libxslt1-dev` specifically, the build hard-failed on the HTTP XSLT module despite ModSecurity never
needing XSLT support at all. Confirmed the exact flag set live via `ssh panel-dev "nginx -V"` before
fixing it.

## The bug that made the WAF a no-op everywhere until today

Found during a real end-to-end verification pass, not from documentation: every SQLi/XSS payload
against a site explicitly set to Enabled came back `200`, not `403`. The WAF *looked* fully healthy
— engine installed, health check green, `nginx -t` clean — but was silently inspecting nothing on
any real site.

Root cause: the internal diagnostic vhost the install script writes for the health check's own
synthetic-attack probe declared its own explicit `modsecurity_rules_file`, on top of the *same* file
`NginxSettingsService` also loads into nginx's `http{}` context once `waf.installed` is set.
ModSecurity-nginx compiles each declaration into its own independent ruleset object rather than
simply inheriting the parent context's one — so the same CRS rule IDs (`901001` and others) ended up
loaded twice, and `nginx -t` failed with "Rule id: 901001 is duplicated" on every config regeneration
from that point on. `InstallWafEngineJob` only logged the failure; the install run still showed
`succeeded` and `waf.installed` stayed `true`, so nothing about the panel's own state hinted anything
was wrong. The fix: the healthcheck vhost no longer declares its own ruleset file at all — it
inherits the one true copy from `http{}` context, exactly like every real account vhost already does.

## Verified

Live, end to end, on `panel-dev`, against a real disposable test domain. A real SQL injection payload
(`?id=1' OR '1'='1`) and a real XSS payload both returned `403` once the duplicate-rule fix was
deployed and applied; the real homepage kept returning `200` throughout. Detection Only mode: the
same payloads returned `200` but showed up as genuinely *logged* matches (rule 942100/941100 etc.,
`"secrules_engine":"DetectionOnly"`) in the real ModSecurity audit log — flipped back to Enabled,
`403` returned immediately. `WafService::blockedRequests()` confirmed parsing real audit-log entries
end to end. The fail-closed guarantee: renaming the loaded module file made `nginx -t` fail exactly
as designed (module missing, config not reloaded, existing traffic unaffected); restoring the file
made `nginx -t` and a real `systemctl reload nginx` succeed again. `apt-cache policy nginx` confirmed
the version pin holding at priority 1001.

23 new unit/feature tests, full existing suite green, `pint --test` clean. Shipped across several
commits as each from-source build issue was found and fixed live, plus the duplicate-rule-ID fix
found during final verification.
