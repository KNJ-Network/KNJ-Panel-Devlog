# Phase 133 - One Page for Every Alert

Phase 131 built the plumbing: the admin alert service, six new health-alert events, and webhook
formats to carry them anywhere. What it did not build was one place to manage it all. Recipient email
and per-event toggles lived on the Server Setup page, and connected webhook platforms lived on a
separate page under Security. Feedback showed a single Notifications page was needed.

## What's new

A single **Servers, Notifications** page (a read-only aggregator) replacing both old locations:

- **Recipients**: primary and secondary contact email, unchanged fields, saved through the existing
  contacts route.
- **Notification types, grouped by severity**: Critical (WAF degraded, backup failed, server
  unreachable, licence issues), Warning (SSL issuance failure) and Info (panel and system update
  available), each group visually distinct.
- **Connected Platforms**: the Slack, Discord and generic webhook form and table, moved here from the
  old standalone Webhooks page.

None of the underlying save logic changed; every form posts to routes that were already tested and
working. This is consolidation, not a rewrite, in the same pattern as the Server Setup plus Server
Settings merge and the Import Account plus Transfer Tool merge.

## The gaps building this surfaced

Putting every alert on one page exposed that SSL-issuance-failure alerts predated the alert service
and had drifted: they used a bespoke notification path with no secondary recipient, no admin fallback
and no webhook fan-out. They now use the alert service like everything else, and the old notification
class is gone.

The second gap: a placeholder saying "defaults to the admin's email" was cosmetic only. The alert
service runs from queued jobs and scheduled commands with no logged-in user, so nothing would have
been delivered if no contact email was saved. The recipient lookup now really falls back to the first
admin account's email.

## Verified

9 new tests for the page plus 2 for the alert service fallback; webhook and server-setup tests
updated for the retired page. Full local suite green and code-style clean.

Live on the dev server: the Notifications page appears and loads, the placeholder hint shows the
admin's email, a toggle round-tripped through the live save route, and a webhook added through the
page's form appeared in the table and was then removed.
