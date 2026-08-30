# Phase 168 - Closing the Loop on the Body-Size Fix

The nginx fix from a few hours earlier (Phase 166) worked — but only because it was applied to `ws1`
by hand, through Server Setup's "Set Hostname & Issue Certificate" button. That raised an obvious
question: what happens to the *next* customer who's already running an older build when this same
class of fix ships? Nothing, it turns out. Silently nothing.

## Why the fix didn't travel on its own

The panel's own controller (2087) and account (2083) vhost is built in exactly two places:
`bootstrap-server.sh`, which runs once at install, and the `issue-panel-cert` provisioning action,
which only re-runs when an admin explicitly resubmits a hostname/cert change. A routine "Update Now"
resyncs the *script file* containing the new template to disk — confirmed live, the panel update log
literally says "Resynced knjpanel-provision-account (changed in this release)" — but resyncing a file
isn't the same as re-executing it. The live nginx config just keeps running whatever template was
generated the last time either of those two things actually happened, indefinitely, until an admin
happens to go re-click a button that has nothing to do with the fix they're waiting on.

This wasn't a new problem invented today. `NginxSettingsService`'s customer-facing snippet had the
identical failure mode once, and already has the fix: `ApplyPanelUpdateJob` calls
`applyCurrentSettings()` after every successful self-update, specifically so a change to what that
snippet generates reaches an already-provisioned server without anyone lifting a finger. The panel's
own vhost template was just never wired into that same mechanism.

## Splitting issue and reapply apart

`issue-panel-cert` does three things bundled together: call certbot, rewrite the vhost, and (for a
first-time hostname) run the full mail+DNS server setup. Only the middle piece needed to run on every
update — calling certbot on every deploy would be actively wrong (rate limits, unnecessary ACME
traffic), and the mail+DNS setup is explicitly gated to first-run only already.

Pulled the vhost-generation logic — role detection, the Roundcube/phpMyAdmin location blocks, the
main heredoc, the webmail/account/controller shortcut vhosts, `nginx -t`, the reload — out into a
single `write_panel_vhost()` function. `issue-panel-cert` calls it after a real certbot issuance, same
as before. A new `reapply-panel-vhost` action calls it standalone, using whatever certificate is
already on disk, no certbot involved — and skips cleanly (not a failure) if no certificate exists yet,
since a server that's never been through Server Setup has nothing to reapply.

`ApplyPanelUpdateJob` now calls this after every successful update, in its own `try`/`catch` sitting
alongside the existing nginx-snippet reapply — deliberately isolated from each other, since they
regenerate two unrelated artifacts and one failing should never suppress the other from even being
attempted.

## The other half: some feedback during the wait

Separately, while testing the original upload bug live, the only sign anything was happening during a
~975MB upload was the browser tab's own loading spinner — easy to mistake for a hang on a slow
connection. Added a small shared partial (`upload-progress-overlay.blade.php`) to both cPanel-import
forms: intercepts the form submit, uploads via `XMLHttpRequest` instead of a native POST specifically
for `xhr.upload.onprogress` (fetch has no cross-browser equivalent for tracking request-body, as
opposed to response-body, progress), and drives a spinner plus a real percentage/byte-count progress
bar. Deliberately sends no `X-Requested-With` or `Accept: application/json` header — Laravel's
`expectsJson()` would otherwise treat the request as AJAX and return a raw JSON 422 on a validation
failure instead of the usual redirect-back-with-flashed-errors, which is what the completion handler
relies on (`xhr.responseURL` after XHR transparently follows the redirect, whichever page it lands on).

## Verifying without a real multi-hundred-MB file

Confirmed live on `panel-dev`, after autodeploy picked up the push: the overlay markup, progress bar,
and `KnjUploadProgress.attach()` wiring are all present and initialize without console errors on the
real page. A full drag-and-drop interaction test needs a real file picker, which the automation
available couldn't drive without extra setup — the next real large-file upload attempt exercises this
exact path end to end regardless.

Full suite (2571/2571) and `pint --test` before any of this shipped. Released as v0.17.19, deployed to
all 4 production servers via Panel Updates — which, fittingly, is the first real-world exercise of the
auto-reapply fix this phase built.
