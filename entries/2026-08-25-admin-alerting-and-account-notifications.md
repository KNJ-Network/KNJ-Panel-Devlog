# Phase 131 - Alerting: The Part That Was Missing From Every Health Check

Audit #15 (Phase 130) closed out with the panel clean. Before the next release, one gap kept
surfacing across almost every health-adjacent feature: detection without delivery. WAF health checks,
scheduled backups, licence validation, linked-server reachability and panel update checks all
recorded their result somewhere and then stopped, so an admin only found out by opening the relevant
page. Separately, the account Preferences page's two notification checkboxes (disk usage running high,
SSL certificate about to expire) had been saved to the database with nothing behind them.

Both gaps share the same shape: state was computed correctly but never reached anyone. They were fixed
together as one alerting pass.

## One admin alert service: one call site, two channels

Every admin-facing health signal now goes through a single alert service: email to the configured
primary and secondary server contacts, and the existing Webhook system, each gated independently.

Wired into five places:

- **WAF health check**: alerts on a healthy-to-degraded transition, with recovery language on the way
  back, not on every poll while degraded.
- **Scheduled backups**: a failed-backup alert from both the per-account and panel-database paths.
- **Licence revalidation**: an alert on a valid-to-invalid transition, plus an expiry warning guarded
  by the expiry value already warned about, so a renewal re-arms it with no reset step.
- **Linked-server health check**: unreachable and recovered alerts on genuine transitions. Our first
  draft would have alerted for a brand-new server still pending its first check; it was narrowed
  before any test was written.
- **Panel update check** (new, daily): an update-available alert deduplicated by version. The check
  had existed but had no scheduled trigger.

All four health-check toggles got their own checkbox on the Server Contacts card.

## Webhooks: two more things they can now do

While extending the event vocabulary, it became clear that the webhook payload format was the real
limiter: a signed generic envelope suits custom integrations but not Slack or Discord. The webhook
job now builds the body by format (generic, Slack or Discord), so a real Slack or Discord webhook URL
just works. Its lack of automatic retry is a deliberate decision and was left alone.

## Account notifications: the long-dead checkboxes

An hourly check runs over every active account with either preference on. Disk usage uses a state
flag that clears when usage drops back under 90%, so a real re-crossing warns again. SSL expiry is
guarded by the exact expiry already warned about, so a renewal re-arms the 14-day warning. Both send
to the contact email and the second contact email when set.

## Verified

10 new tests for account notifications plus earlier tests for the alert service, webhook formats and
each integration. Full local suite green (1,947 tests) and code-style clean.

Live on the dev server: three disposable webhooks, one per format, pointed at a local listener,
received correct per-format bodies with valid signatures when a real alert fired, and the email side
reached the mail server. All test webhooks were cleaned up afterward.
