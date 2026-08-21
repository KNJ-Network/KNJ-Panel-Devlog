# Phase 109 - Mail Only Stage 3 Continues: the Rest of the Dispatcher Migration

Followed Phase 108's webmail fix with the bulk of the remaining work: migrating every mail-related
service still bypassing the Stage 2 dispatcher. Four releases (v0.16.32 through v0.16.36), each
shipped, deployed, and live-verified against the real linked satellite before starting the next.

## Tier A: 12 services that were calling the script directly

Greylisting, spam scoring, SMTP restrictions, mail filters, relay config, server config, mailbox
permission repair, sieve scripts, challenge-response mail, and the usage report generator all
built their own arguments and called the privileged provisioning step directly. All twelve now go
through the shared runner.

Nine of them also carried their own temp-file validation that did not match what the runner
produces. That mismatch is invisible locally and only fails when an action is dispatched to a
satellite. All nine were fixed in the same commit as their conversion, because Phase 108 had just
shown how easily this class of "works locally, breaks over the wire" bug hides from anything short
of a real dispatch. One service also stopped reading a local config file to answer "has this been
applied" and now uses the flag Main tracks in its own database.

## Tier B: mail queue and delivery logs had no script action at all

These services had nothing to bypass, calling local commands directly. Four new actions (queue
list, retry, retry-all, log read) were added and the existing queue-delete actions were routed
through the runner. The log-read action needed to treat "no matches" as a normal result, since the
runner does not expose exit codes; without that, every zero-match search would report a failed
dispatch.

## Tier C: catch-alls and mailing lists had the mailbox problem, one level up

Stage 2's auth-map service solved this for mailboxes, but catch-alls and mailing lists use the same
lookup mechanism and were not covered. Two sibling services now push their own flat maps, are wired
into the existing mutation paths, and replay on server switch-over. Plain forwarders were the one
feature still without a push, and the documentation said so.

## Verified

Full local suite (1644 tests) and the code-style check green after every tier. After each one we
cut the release, upgraded the linked satellite via its own Panel Updates page, dispatched a real
action from Main and confirmed the satellite's state matched: a relay config write, a sieve
compile, a queue list, a log search (including a deliberate zero-match query), and a catch-all plus
a mailing list with two members.
