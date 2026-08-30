# Phase 167 - The Save That Ate Its Own Response

WHM PHP Settings writes one shared php.ini override that every PHP-FPM pool on the box loads —
hosting accounts and the panel's own admin pool alike. Saving the form on the real dev server
confirmed the write itself was correct: the right values landed in
`/etc/php/8.5/fpm/conf.d/99-knjpanel-global.ini`, and the privileged provisioning script's reload
of `php8.5-fpm` succeeded. And yet the browser got a 502 every time.

## Reloading the pool serving your own request

The reload target is `php8.5-fpm`, unqualified — every pool on the box, including whichever one is
currently running the admin's own POST request to save the form. `PhpSettingsService::apply()`
called the provisioning script (write the ini, validate with `php-fpm8.5 -t`, `systemctl reload
php8.5-fpm`) synchronously, then returned control to the controller, which only then persisted the
`php.*` Setting rows the WHM page reads back on next load. The reload tore down the very worker
generating the response before any of that could finish — the browser saw a 502, and because the
Setting writes never got a chance to run, the WHM page came back showing stale values on next load
despite the real php.ini already holding the new ones.

Nginx settings hit the same "write, then persist Settings" shape and don't have this problem —
reloading nginx doesn't kill the PHP-FPM worker producing the response, so the two only look
identical from the Laravel side.

## The fix: split the write from the reload

The provisioning script's `php-settings-write` action used to do three things in one shot: back up
the current ini, write and validate the new one, then reload. It now stops after validation —
reload is a new, separate no-argument action, `php-settings-reload`. `PhpSettingsService` gained a
matching `reload()` method, and the controller schedules it via `App::terminating()` instead of
calling it inline. Laravel's own response-send path already calls `fastcgi_finish_request()` before
`Kernel::terminate()` runs — that's the one existing hook in the framework that's guaranteed to fire
only after the response has already left for the browser, so no new plumbing was needed to get the
ordering right.

Net effect: the write-and-validate step (never itself the actual problem) still runs synchronously
and still gates persisting the Setting rows, exactly as before. Only the live reload — the one step
that can legitimately compete with the current request for the worker running it — moves to run
after the response is gone. A reload failure at that point can only be logged, not shown to the
admin, but by then the ini it's reloading has already passed `php-fpm8.5 -t`, so there's nothing
left for it to fail on short of the service itself being down.

Two new regression tests, plus a strengthened assertion on the existing write-failure test: the new
ones confirm both the write and the now-separate reload action actually run, and that a reload
failure after the fact doesn't turn a successful save into an error response or drop the persisted
settings; the strengthened one confirms a *write* failure (the case that already worked correctly)
still never schedules a reload at all. Tested (2565/2565).

Released as v0.17.18.

## Round 2 - The fix that still 502'd once

Live-testing this exact fix on panel-dev — save the form, confirm the Setting rows landed, confirm
no 502 — the *save request itself* came back clean. Then the browser did what browsers do after a
303 redirect and auto-followed it, and that follow-up GET came back `502 Bad Gateway`. Refreshing a
second later loaded fine, and the saved values were correctly there — so `App::terminating()` had
fixed the exact bug reported (the admin's own response no longer got eaten, and the Setting rows no
longer got lost), but a *new*, narrower window opened up right behind it.

The reason: reloading `php8.5-fpm` restarts every pool on the box at once, not just the worker that
happened to call for it. `App::terminating()` runs in the same PHP-FPM worker process, in the same
instant `fastcgi_finish_request()` hands the response off to nginx — there is no gap between "the
save response is gone" and "the pool-wide restart begins." The browser's auto-follow GET fires within
milliseconds of receiving the redirect, which is more than enough to land squarely inside that
restart window even though it's a completely different request landing on a (probably) completely
different worker.

Deferring further in the same process can't close a gap that's about which *pool*, not which
*request* — so the reload moved off the request lifecycle entirely: a new `ReloadPhpFpmJob`,
queued (this app already runs `knjpanel-queue.service`, a database-backed `queue:work` process) with
a 3-second delay. The redirect-follow request — and, realistically, the page render after it —
finishes well inside that window on any reasonable connection, so the disruptive part of the reload
never overlaps a request that has anything to do with the save that triggered it. Same
best-effort-and-log failure handling as `InstallWafEngineJob`/`UpdateWafRulesetJob`, the two existing
jobs in this codebase built around the same "somebody has to reload something disruptive later"
shape.

One new job-level test (`ReloadPhpFpmJobTest`) covers `handle()` actually calling the reload action
and a failed reload getting logged rather than thrown; `PhpSettingsTest` swapped its `Process`-based
assertion on the reload for a `Bus::fake()` one confirming `ReloadPhpFpmJob` is dispatched with a
delay and never runs inline. Tested (2572/2572).

Re-verified live on panel-dev, deliberately more rigorously than the first pass that missed the
redirect-follow race — browser-automation clicks and the network-request log turned out to be too
ambiguous to trust here (couldn't reliably tell a real submit from a UI no-op, and stale console
entries survived same-tab navigations). Drove the save directly from the page's own JS console
instead: a `fetch()` POST to the form's own action got back a same-origin redirect (not a raw error),
an immediate follow-up `fetch()` GET came back `200` with the new `memory_limit_mb` value already in
the rendered HTML, and — the part that actually proves the *reload*, not just the write — `ps` showed
every php8.5-fpm pool's worker processes freshly restarted a few seconds later, right on schedule with
the job's delay. Setting row and `99-knjpanel-global.ini` both correctly hold the saved value
afterward, and `storage/logs/laravel.log` has nothing from `ReloadPhpFpmJob`, confirming a clean run.

Released as v0.17.20.
