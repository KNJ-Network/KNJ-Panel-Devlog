# Phase 109 - Mail Only Stage 3 Continues: the Rest of the Dispatcher Migration

Followed Phase 108's webmail fix with the bulk of the remaining work: migrating every mail-related
service that was still bypassing the Stage 2 dispatcher, in the order laid out at the end of that
entry — mail filters, greylisting, relay config, spam settings, mail queue, delivery logs, then
catch-alls and mailing lists. Four releases (v0.16.32 through v0.16.36), each shipped, deployed, and
live-verified against the real linked satellite (`mail.dev.knj.network`) before starting the next.

## Tier A: 12 services that were calling the script directly

Greylisting, spam scoring, SMTP restrictions, mail filters, relay config, server config, mailbox
permission repair, sieve scripts (autoresponder/challenge-response/mail rules all share one writer),
challenge-response pending mail, and the usage report generator all had the same shape: they built
their own args and temp file, then called `Process::run(['sudo', SCRIPT, ...])` straight past the
dispatcher Stage 2 built. Converted all twelve to `ProvisioningScriptRunner::run()`.

Nine of them carried their own bespoke temp-path regex in `knjpanel-provision-account`
(`^/tmp/knjpanel-<label>-...$`) instead of the shared one the Runner's generic temp file actually
produces. That mismatch is silent locally — the file still exists at the path the regex expects, so
nothing complains — and only 403s the moment the same action gets dispatched to a satellite instead
of run in place. Caught and fixed all nine as part of the same commit as their PHP conversion,
specifically because Phase 108's webmail bug had just demonstrated how easily this class of "works
locally, breaks over the wire" bug hides from anything short of a real dispatch.

One of the twelve, `MailServerConfigService::apply()`, also picked up a `Setting` flag
(`mail.server_config_applied`) on success — `SieveScriptWriter::ensureSieveEnabled()` was reading a
local Dovecot config file to answer "has this been applied," which is meaningless on a satellite;
now it reads the same flag Main already tracks authoritatively in its own DB.

## Tier B: mail queue and delivery logs had no script action at all

`MailQueueService` and `MailDeliveryLogService` weren't bypassing the dispatcher — they had nothing
to bypass, calling bare `postqueue`/`tail`/`grep` against whatever's local. Added four new actions
(`mail-queue-list`, `mail-queue-retry`, `mail-queue-retry-all`, `mail-log-read`) and converted the
two pre-existing queue-delete actions to route through the Runner too.

`mail-log-read`'s grep needed a `|| true` that isn't optional: `ProvisioningResult` only exposes
`successful()`/`output()`/`errorOutput()`, no exit code, so the old code's "grep exit 1 means no
matches, not a failure" logic has nowhere to live on the PHP side anymore. Without the `|| true`,
every zero-match search would report as a failed dispatch.

## Tier C: catch-alls and mailing lists had the mailbox problem, one level up

Stage 2's `MailAuthMapService` solved this for real mailboxes: Postfix/Dovecot both expect to query
Main's live database directly to answer "does this address exist," and a satellite has no copy of
that database worth querying, so the service pushes flat files instead. Catch-alls and mailing
lists share the exact same `virtual_alias_maps` mechanism and the exact same problem, but weren't
covered — `setup-mail-server.sh` correctly repoints mailbox delivery at flat files on a `mail_only`
install, but leaves `virtual_alias_maps` pointed at a local `mysql:` config with nothing behind it.
Silent no-op, same shape as everything else in this stage.

Built two sibling services, `DefaultAddressAuthMapService` and `MailingListAuthMapService`, each
pushing their own flat `hash:`-mapped file the same way. Wired into
`DefaultAddressService`/`MailingListService`'s existing mutation methods and into the mail-server
switch handler, so linking a fresh satellite replays existing catch-alls and mailing lists on
switch-over, not just mailboxes.

Still not covered: plain forwarders (`MailForwarder`), the one remaining feature on this same
`virtual_alias_maps` mechanism without its own push — flagged directly in `MailAuthMapService`'s doc
comment as the next gap in this family, not silently left out.

## Verified

Full local suite (1644 tests) and `pint --test` green after every tier. Live, for real, after each
one: cut the release, upgraded `mail.dev.knj.network` via its own Panel Updates page, then
dispatched a real content-bearing action from Main against the satellite and confirmed the actual
file/state on the satellite matched — a relay config write, a sieve script compile, a queue list, a
log search (including a deliberate zero-match query, to prove the masked grep exit code behaves),
and finally a real catch-all address plus a real mailing list with two members, confirming both
`/etc/postfix/virtual-catchall-map` and `/etc/postfix/virtual-mailinglist-map` matched exactly on
the satellite before cleaning up.

## Next

Service Status (the dedicated page and the dashboard widget) still has no concept of a linked mail
server — it's local-`systemctl`-only, so it keeps reporting Main's own now-irrelevant Postfix/
Dovecot state once mail is running elsewhere. That's the last piece of Stage 3, followed by a full
end-to-end verification pass across everything this stage built.
