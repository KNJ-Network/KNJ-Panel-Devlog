# Phase 162 - The Save Button Nobody Knew They Had to Click

Shipping last phase's webmail/account proxy fix surfaced a smaller problem hiding underneath it: the
generated nginx snippet every account vhost includes only ever gets rewritten when an admin visits
Server Configuration → Nginx Settings and clicks Save & Apply. Upgrading the panel's own code doesn't
retroactively rewrite it. `NginxSettingsService::buildSnippet()`'s output had just changed — three
`return 301` lines became `proxy_pass` blocks — but every already-provisioned install kept serving the
old, pre-upgrade config until someone happened to click that button by hand. Confirmed live, twice, on
two different servers, in the same session that shipped the fix it was hiding.

## The fix already had a name

`NginxSettingsService` already has `applyCurrentSettings()` — no arguments, rebuilds the snippet from
whatever's currently persisted rather than a specific value a form just submitted. It exists for
exactly this shape of problem: `InstallWafEngineJob` already calls it the moment a WAF install finishes,
so the new `modsecurity` directives show up immediately instead of waiting for an unrelated settings
change to trigger a rebuild. The panel's own self-update just needed the same treatment.

`ApplyPanelUpdateJob::handle()` now checks whether `PanelUpdateApplyService::apply()` left the run
`Completed`, and if so, calls `applyCurrentSettings()` — same try/catch shape as the WAF job, so a
failure there gets `Log::error`'d but can never turn an otherwise-successful update into a failed one.
No role guard needed either: nginx ships on every install regardless of role, and every other caller of
`applyCurrentSettings()` in this codebase already runs unconditionally the same way.

## Proving it, and hitting an honest wall

Bumped to v0.17.13, cut the release, clicked Update Now on `ws1`. Watched `journalctl` for the
`nginx-settings-write` call that should follow. It never showed up — not as a failure, just absent
entirely, like the new code had never run.

It hadn't. The queue worker that picked up this update had already loaded `ApplyPanelUpdateJob` into
memory *before* the update it was running rewrote that very class on disk. `knjpanel-upgrade` restarts
the worker as its own last step — the log confirmed it, `knjpanel-queue-restart.timer` firing about
seven seconds after the update itself had already finished and logged `DONE`. The worker that executed
this update was running yesterday's code the entire time it was installing today's.

That means this exact self-heal step can never prove itself on the update that introduces it — only
from each server's *next* self-update onward, once the worker that picked it up is one that already
restarted with the new class loaded. It's the same shape as the bulk-update-all bootstrap gap from two
phases ago: a feature whose first real activation is structurally deferred by one cycle, not a defect
in what shipped. Confirmed the worker actually did restart with the new code in memory (the
`knjpanel-queue-restart.timer` log line proves it), and leaned on the five dedicated tests added
alongside the fix — success path, failure-doesn't-downgrade, and does-not-run-on-a-failed-update — to
cover the part a single live run couldn't.

Rolled out to all three satellites via the bulk button afterward. All four servers landed on v0.17.13
clean, real homepage still returned 200. From here on, every self-update on every one of them re-applies
Nginx Settings on its own, no Save & Apply click required.

Tested (2546/2546).
