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
