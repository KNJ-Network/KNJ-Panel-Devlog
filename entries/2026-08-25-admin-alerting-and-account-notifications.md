# Phase 131 - Alerting: The Part That Was Missing From Every Health Check

Audit #15 (Phase 130) closed out with the panel clean and ready to source a production release. Before
touching that release — the old fleet is being retired for a fresh Main/Mail Only/two-DNS-Only build —
one gap kept surfacing across almost every health-adjacent feature shipped so far: detection without
delivery. WAF health checks, scheduled backups, licence validation, linked-server reachability, and
panel update checks all wrote their result *somewhere* (a `Setting` flag, a model status column, a log
line) and then stopped. An admin only found out by opening the relevant page. Separately, the account
Preferences page's two notification checkboxes — "notify me when disk usage is running high" and
"...when an SSL certificate is about to expire" — had saved to the database since Phase early-on with
nothing behind them at all.

Both gaps share the same shape: state was already being computed correctly, it just never reached
anyone. Fixed together as one alerting pass.

## AdminAlertService: one call site, two channels

Every admin-facing health signal now funnels through `AdminAlertService::send()`: email to whichever of
`server.contact_email`/`server.contact_secondary_email` are configured (the secondary field existed
before this but was never actually used to send anything), and the existing Webhook system, both gated
independently — the email side by a per-alert-type `Setting` toggle the caller passes in, the webhook
side entirely by each webhook's own `enabled` + subscribed-`events` state.

Wired into five places:

- `CheckWafHealth` — fires `waf.degraded` only on a healthy→degraded transition, `waf.recovered`-shaped
  language on the way back, not on every five-minute poll tick while still degraded.
- `BackupService::runScheduled()` — `backup.failed`, from both catch blocks (per-account and the
  panel-DB backup), traced first to confirm `backupAccount()` genuinely re-throws on failure rather than
  swallowing it internally, so the alert placement actually fires for the real failure path.
- `LicenceService::revalidate()` — `licence.issue` on a valid→invalid transition, plus a new
  7-day-ahead expiry warning guarded by `Setting('licence.expiry_alerted_for')` storing the exact expiry
  value already warned about, so a renewal (which always changes that value) re-arms the check with no
  separate reset step needed.
- `ServerHealthCheckService::check()` — `server.unreachable` / `server.recovered` on a genuine
  Online→Offline or Offline→Online transition. Caught one bug in my own first draft here: the initial
  condition (`$previousStatus !== Offline`) would have fired an "unreachable" alert for a brand-new
  server sitting at the normal `Pending` status on its very first health check, before it's even fully
  linked. Fixed to the narrower `$previousStatus === Online` before any test was written against it.
- `CheckPanelUpdateAvailable` (new, daily) — `panel.update_available`, deduped by version so it never
  re-alerts for a release an admin has already seen. `PanelUpdateCheckService::check()` had existed for
  a while but had no scheduled trigger at all — an update could sit available for weeks with nothing
  running the check in the background.

All four health-check toggles (WAF/backup/licence/server-unreachable) got their own checkbox on the
existing Server Contacts card, independently switchable.

## Webhooks: two more things they can now do

While extending the event vocabulary for the alerts above (`waf.degraded`, `backup.failed`,
`licence.issue`, `server.unreachable`, `server.recovered`, `panel.update_available`, alongside the
existing account lifecycle events), it became obvious the webhook payload shape itself was the real
limiter — a signed `{event,timestamp,data}` envelope is exactly what a custom integration wants and
exactly what Slack or Discord's incoming-webhook endpoints don't understand. `SendWebhookJob` gained a
`buildBody()` step that branches on the webhook's own `format`: `generic` (unchanged), `slack`
(`{"text": "*subject*\nmessage"}`), or `discord` (`{"content": "*subject*\nmessage"}`) — so pasting a
real Slack or Discord webhook URL into the existing Webhooks page now just works, no separate
integration to build. (Checked `SendWebhookJob`'s own doc comment before assuming its lack of automatic
retry was a gap to fix — it's a deliberate decision, blindly retrying a dead endpoint forever isn't
useful, left alone.)

## AccountNotificationService: the two-year-old dead checkboxes

`checkAll()` runs hourly over every active account with either preference on. Disk usage: a state-machine
flag (`disk_usage_warning_sent_at`) that clears itself the moment usage drops back under 90%, so a real
re-crossing (grew, got cleaned up, grew again) warns again each time rather than going silent forever
after the first warning ever sent. SSL expiry: guarded by the exact expiry timestamp already warned
about rather than a boolean, so a renewal — which always changes `ssl_expires_at` — naturally re-arms the
14-day warning with no separate reset step, same pattern as the licence-expiry guard above. Both send via
the existing on-demand `Notification::route('mail', ...)` pattern to both `contact_email` and
`second_contact_email` when set.

## Verified

10 new tests for `AccountNotificationService` (crossing/staying/dropping/re-crossing the disk threshold;
sending/not-resending/re-arming SSL expiry; preference-off skip; dual-recipient delivery), plus the
earlier tests for `AdminAlertService`, the three webhook formats, and each of the four health-check
integrations. Full local suite green (1,947 tests) and `pint --dirty` clean before deploy.

Live on `panel-dev`, against the real deployed code (not a local re-run): created three disposable
webhooks — one per format — pointed at a throwaway local HTTP listener, then called the real
`AdminAlertService::send()` through `tinker` to fire a genuine `waf.degraded` alert. Confirmed the queue
worker delivered all three real HTTP requests with the correct per-format body (`{"event":...,"data":
{...}}` for generic, `{"text":"*subject*\n..."}` for Slack, `{"content":"*subject*\n..."}` for Discord),
each carrying a valid `X-KNJ-Signature`. Confirmed the email side genuinely reached Postfix too — the
mail log shows the alert relayed and accepted (`status=sent`) to the configured admin address, not just
queued. All three test webhooks and the log/listener process cleaned up afterward.
