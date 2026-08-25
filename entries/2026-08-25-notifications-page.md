# Phase 133 - One Page for Every Alert

Phase 131 built the plumbing — `AdminAlertService`, six new health-alert events, and the webhook
delivery formats to carry them anywhere. What it didn't build was a place to actually *manage* any
of it: recipient email lived on the Server Setup page's Server Contacts card, the per-event toggles
lived there too, and connected webhook platforms lived on a completely separate standalone page
under Security. Asked directly whether a dedicated Notifications page existed for managing all of
this together — it didn't. It does now.

## What's new

A single **Servers → Notifications** page (`NotificationsController::index()`, GET-only) replacing
both of the old locations:

- **Recipients** — primary/secondary contact email, unchanged fields, still saved through the
  existing `server-settings/contacts` route.
- **Notification types, grouped by severity** — Critical (WAF degraded, backup failed, server
  unreachable, licence issues), Warning (SSL issuance failure), Info (panel/system update
  available), each group visually distinct (red/amber/sky badges) instead of one flat list of
  checkboxes.
- **Connected Platforms** — the Slack/Discord/generic webhook add-form and table, moved here
  wholesale from the old standalone Webhooks page.

None of the underlying save logic changed. The new controller is a read-only aggregator; every form
on the page posts to the same routes that were already tested and already working
(`ServerSettingsController::updateContacts()`, `WebhookController::store/update/test/destroy`) —
consolidation, not a rewrite. Same pattern as the Server Setup + Server Settings merge and the
Import Account + Transfer Tool merge before it.

## The gap building this surfaced

Putting every alert type on one page made an inconsistency visible that wouldn't have been obvious
looking at any single file in isolation: SSL-issuance-failure alerts predated `AdminAlertService`
and had drifted from it. `ProvisionAccountJob` was firing a bespoke direct
`Notification::route('mail', ...)->notify(new SslIssuanceFailed(...))` call — no secondary-recipient
support, no admin-email fallback, and it bypassed the webhook fan-out every other alert type gets.
It would have shown up on the new page's severity list looking identical to the other five events
while actually behaving differently underneath. Unified it onto `AdminAlertService::send()` like
everything else; `SslIssuanceFailed` is gone.

The other gap: a UI placeholder showing "defaults to the admin's email" isn't the same as an actual
runtime fallback. `AdminAlertService` fires from queued jobs and artisan commands with no
`auth()->user()` in scope, so a cosmetic hint alone would never actually deliver anything if an
admin never got around to saving a contact email. `recipientEmails()` now falls back to
`User::where('role', Role::Admin)->orderBy('id')->value('email')` for real, not just in the view.

## Verified

9 new tests in `NotificationsControllerTest` (page renders both sections, severity groups and their
toggle names all present, a connected webhook shows by name, saving through the existing contacts
route persists correctly, admin's own email shows as the placeholder hint, nav placement, the old
webhooks GET route now 405s instead of rendering, non-admin and reseller both forbidden), plus 2 new
`AdminAlertServiceTest` cases for the admin-email fallback and its priority against an explicit
contact email. Updated `WebhookControllerTest`/`ServerSetupTest` for the retired page/card. Full
local suite green, `pint --dirty` clean.

Live on `panel-dev`: confirmed "Notifications" appears under the Servers nav group and the page
loads with both sections. Confirmed the admin's own email shows as the contact-email field's
placeholder when none is configured. Round-tripped a real toggle (SSL-issue alerts off, then back
on) through the live save route and confirmed each state via `Setting::get()` over SSH before and
after, leaving the real configured contact email untouched throughout. Added a real webhook through
the page's embedded Connected Platforms form — a genuine native form submit, not simulated — and
confirmed it landed correctly in the table with the right format and subscribed event; removed it
afterward.
