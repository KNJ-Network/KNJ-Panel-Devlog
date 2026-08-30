# Phase 169 - The Upload That Vanished

Two fixes shipped tonight already made the ~975MB real-world cPanel migration test possible — nginx
and PHP would no longer reject the file outright, and a fresh spinner + progress bar would prove the
upload was actually moving. Both worked. The upload finished. And then the page came back completely
blank: no error, no new account, no sign anything had happened at all.

## Reading the server's own logs to find out what actually happened

Nothing in Laravel's log. Nothing in nginx's error log for that time window. The only way to find the
real story was the plain nginx access log, read line by line:

```
POST /controller/accounts/import-cpanel  302  506
GET  /controller/accounts/import-cpanel  200  12278
GET  /controller/accounts/import-cpanel  200  12171
```

Three requests, one second apart. The POST really did complete — a 302, not a timeout or a dropped
connection. Then two GETs to the identical URL, 107 bytes apart. That gap is the whole bug.

## What the size difference was actually hiding

The upload overlay's completion handler did `window.location.href = xhr.responseURL` once the XHR
finished. The mistake: XHR follows redirects transparently and automatically, which means by the time
`load` fires, the browser has *already* made its own internal GET to the redirect target — the 12278-
byte response above, carrying Laravel's flashed validation/staging error and the resubmitted form
values. That's a one-shot read: the framework clears flashed session data as soon as a request
consumes it. The second GET — the one `window.location.href` triggers — arrives a request too late,
finds nothing left to flash, and renders the pristine 12171-byte form instead. The real error message
that was supposed to end the mystery was fetched, rendered once inside the XHR's own internal request,
and then thrown away.

## The fix

`xhr.responseText` already holds that fully-rendered final page — errors, resubmitted values, all of
it. There was never a need to fetch it again. Swapped the extra navigation for rendering what XHR
already has:

```js
history.replaceState(null, '', xhr.responseURL);
document.open();
document.write(xhr.responseText);
document.close();
```

`document.write` is unfashionable, but it's the one approach here where inline `<script>` tags in the
fetched page actually execute — an `innerHTML` swap would leave them inert.

Verified live on `panel-dev`: submitted a deliberately invalid archive through the real form, and
`Not a valid .tar.gz archive.` now appears exactly where it should, instead of a form that resets
itself back to blank as if nothing had been typed.

## The other question that came up mid-diagnosis: does any of this leave anything behind?

With no server-side record of what the real failure was, the obvious next worry was whether a ~975MB
temp file or a half-extracted archive was still sitting on `ws1`'s disk. It wasn't, and by design:
`CpanelImportService` deletes the copied upload in a `finally` block regardless of outcome, and removes
the extracted staging directory both on a successful import and on any caught failure. The one real gap
found while checking this: that cleanup only caught `RuntimeException`, not `Throwable` — so a genuinely
unexpected error (not one of the service's own validation checks) could have skipped it and leaked an
extracted directory. Widened both staging methods to `catch (Throwable)`, matching the pattern
`runImport()`/`runImportIntoAccount()` already used for the same reason.

Full suite (2572/2572) and `pint --test` before shipping. Released as v0.17.21, deployed to all 4
production servers.
