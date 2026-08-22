# Phase 112 - Mail Only, fully closed out

Three loose ends left over from Stage 3/4, all the same shape: a feature that worked correctly on
Main but never got wired through `ProvisioningScriptRunner`, so it silently kept acting against Main
even once a Mail Only satellite took over.

## Plain forwarders

Catch-alls and mailing lists got their own satellite auth-map push in Stage 3.4; plain forwarders
(`MailForwarder`) were the one remaining `virtual_alias_maps`-backed feature without one —
`MailAuthMapService`'s own doc comment flagged this explicitly as the next gap. A new
`ForwarderAuthMapService` follows the exact shape of its two siblings: no-op unless a Mail Only
server is active, otherwise pushes the full current forwarder set as a Postfix hash map via a new
`mail-forwarder-map-write` provisioning action (self-registers into `virtual_alias_maps` on first
write, same as the catch-all/mailing-list maps before it). Wired into
`MailboxService::createForwarder()`/`deleteForwarder()` and folded into
`MailSettingsController::updateServer()`'s existing resync-on-switch alongside the other two. Before
this fix, a forwarder created while mail was on a satellite stored fine in Main's database but
produced zero Postfix config on the satellite — that mail silently bounced, with nothing in the UI
suggesting anything was wrong.

## DKIM

Turned out not to be a missing feature at all. OpenDKIM installation, milter wiring, and per-domain
keypair generation were already fully built, and already installed on Mail Only satellites via
`setup-mail-server.sh`. The bug was much smaller: `DnsZoneService::generateDkimKey()`/`removeDkimKey()`
called the provisioning script directly via `Process::run` instead of going through
`ProvisioningScriptRunner` the way every other mail-touching service does — so the key always
generated on Main regardless of where Postfix actually ran. One-line fix per method: route through
the runner instead.

## Mail statistics

The WHM "Mail Statistics" page (`MailStatisticsService`, sent/relayer summaries) read
`/var/log/mail.log` straight off Main's own filesystem via a direct `Process::run(['tail', ...])`,
so it silently showed empty or stale data once mail moved to a satellite — the same class of bug as
mail delivery-log reads had before `MailDeliveryLogService` got fixed. Routed through the same
`mail-log-read` action that service already uses; its no-query behavior already tails exactly 5000
lines, matching this service's prior default, so nothing else needed to change on the provisioning
side.

## Verified

Full local suite (1678 tests) and `pint --test` green. Live, for real: cut v0.16.39, upgraded
`mail.dev.knj.network` via its own "Update Now" (Main's own git-autodeploy already had it). Created
a real `livetest-fwd@test.knj.network` forwarder from Main — SSH'd into the satellite and confirmed
`/etc/postfix/virtual-forwarder-map` had the entry, registered in `virtual_alias_maps`, resolvable
via `postmap -q`. Generated a real DKIM key for `test.knj.network` from Main — landed under
`/etc/opendkim/keys/` on the satellite with correct `KeyTable`/`SigningTable` entries, while Main's
own (unrelated, pre-existing) DKIM directory stayed untouched, proving the call actually reached the
satellite. Read mail statistics from Main and confirmed the content matched the satellite's real
`mail.log` line-for-line, not Main's own irrelevant log. All test resources cleaned up afterward —
forwarder deleted (satellite map confirmed empty again), DKIM key removed (satellite directory and
`KeyTable` entry confirmed gone).

This closes out every open item tracked under the Mail Only epic across this session: Stage 1
(linking), Stage 2 (dispatcher), Stage 3 (every remaining mail service migrated), Stage 4
(cross-server SSO), and now this final cleanup pass. Nothing scoped to Mail Only remains open.
