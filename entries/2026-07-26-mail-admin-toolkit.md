# Phase 17 - A Real Mail Admin Toolkit

Same day as the DNS roadmap closing out, the mail side of the Controller area went from "the
basics work" to a genuine admin toolkit — the kind of pages a mail admin reaches for when
something's wrong or needs tuning, not just account/mailbox CRUD.

## Outbound relay (smarthost)

Route this server's outbound mail through a third-party relay instead of sending directly — the
credential is encrypted at rest and never redisplayed after saving.

## Mail server configuration

Which protocols are enabled, TLS policy, and process limits — all editable without touching a
config file over SSH, applied through a fully-managed configuration.

## Mail Queue Manager

Every message currently queued, with the reason a deferred message hasn't gone out yet. Retry one
message or the whole queue, or clear it. Verified against a real deferred message on a development
server.

## Delivery reports and mail statistics

Search the mail log for a specific message's delivery history, or browse recent activity. A
statistics view aggregates sent volume by domain and lists authenticated relay logins — useful for
spotting unusual volume.

## Filter incoming email by domain or country

Reject incoming mail from a specific sender domain, or from an entire country's IP ranges, at the
SMTP level — confirmed with a real blocked connection attempt.

## Greylisting

Temporarily defers mail from senders the server hasn't seen before. Real mail servers retry
automatically within a few minutes; most spam-sending software doesn't. Confirmed live with a real
non-local connection.

## Spam filtering

Content-based scoring, wired in as an additional mail filter alongside the DKIM signing already
running. Verified with the standard industry spam-test string, confirming the message was tagged.

## Repair Mailbox Permissions, Autoresponders, and Email Deliverability

One-click repair when a mailbox's file ownership drifts and mail stops delivering; per-mailbox
autoresponders built on the mail server's own standard extension; and a per-domain check of SPF,
DKIM, DMARC, and reverse DNS, reachable from the same area as the existing nameserver diagnostic.

## The pattern, still holding

Every one of these was tested against a real running development server, not mocked. A few
infrastructure gaps turned up along the way, and each was fixed the same day it was found.
