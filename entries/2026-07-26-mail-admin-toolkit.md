# Phase 17 - A Real Mail Admin Toolkit

Same day as the DNS roadmap closing out, the mail side of the Controller area went from "the
basics work" to a genuine admin toolkit — the kind of pages a real mail admin actually reaches for
when something's wrong or needs tuning, not just account/mailbox CRUD.

## Outbound relay (smarthost)

Route this server's outbound mail through a third-party relay instead of sending directly — the
credential is encrypted at rest and never redisplayed after saving, only ever replaced or removed
explicitly.

## Mail server configuration

Which protocols are enabled, TLS policy and cipher list, and process limits — all editable without
touching a config file over SSH. Applied through one fully-managed configuration file rather than
patching the mail server's own stock config directly, since exact wording in that stock file turns
out to differ between minor versions of the software.

## Mail Queue Manager

Every message currently queued — incoming, active, deferred, or held — with the actual reason a
deferred message hasn't gone out yet. Retry one message or the whole queue, or clear it. Verified
against a real deferred message, not a synthetic one: pointed the relay at an address that can't
be reached, confirmed the real reason showed up in the UI, retried it, then cleared it.

## Delivery reports and mail statistics

Search the mail server's own log for a specific message's delivery history, or browse recent
activity generally. A separate statistics view aggregates sent volume by domain and lists every
authenticated relay login — useful for spotting one domain sending an unusual volume, or a
mailbox that's been compromised and started relaying spam.

## Filter incoming email by domain or country

Reject incoming mail from a specific sender domain, or from an entire country's IP ranges, at the
SMTP level — confirmed with a real blocked connection attempt actually getting rejected, not just
a database flag being set.

## Greylisting

Temporarily defers mail from any sender/recipient/IP combination the server hasn't seen before.
Real mail servers retry automatically within a few minutes; most spam-sending software doesn't
bother. Confirmed live against a real, non-local connection — a loopback test would have silently
bypassed the exact restriction chain being tested.

## Spam filtering

Real content-based scoring, wired in as an additional mail filter alongside the DKIM signing that
was already running. Verified with a real external connection delivering the standard
industry-wide spam-test string, confirming the message actually got tagged and its subject
rewritten, not just that a setting was saved.

## Repair Mailbox Permissions, Autoresponders, and Email Deliverability

Rounding out the toolkit: one-click repair when a mailbox's file ownership drifts and mail stops
delivering; real per-mailbox autoresponders (not a bespoke reimplementation — built on the mail
server's own standard autoresponder extension); and a per-domain check confirming SPF, DKIM,
DMARC, and reverse-DNS are all actually configured correctly, reachable from the same area as the
existing nameserver diagnostic tool.

## The pattern, still holding

Every one of these was tested against the real running server, not mocked — a real deferred
message, a real spam string, a real non-loopback SMTP connection, a real wiped zone. A few
genuine infrastructure gaps turned up along the way (a couple of config directories the
application didn't yet have write access to, one socket path that only resolves correctly once you
account for how the mail server sandboxes its own processes) — each one fixed the same day it was
found, not deferred.
