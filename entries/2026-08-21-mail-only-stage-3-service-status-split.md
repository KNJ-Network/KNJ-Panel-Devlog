# Phase 110 - Mail Only Stage 3 Wraps Up: Service Status Stops Lying About Where Mail Runs

Closed out the last piece of Stage 3: Service Status — both the dedicated admin page and the
dashboard widget — had no idea mail might be running on a linked satellite at all. It's local-
`systemctl`-only, so once an admin activates a Mail Only server, both kept reporting Main's own now-
irrelevant Postfix/Dovecot state instead of anything that matters.

## The overlay, not a rewrite

`ServiceStatusService` didn't need to change — "what's running on this box" is still exactly what it
should answer. The satellite-awareness lives one layer up: a new `MailServiceStatusService` overlays
just the `postfix`/`dovecot` rows of whatever the local service already returned, with the active
satellite's real state. A new no-args dispatcher action, `mail-service-status-report`, emits compact
`UNIT:postfix|ACTIVE:1` lines using the *exact same* `systemctl is-active` check the local service
already runs — deliberately, so "active" never means two different things depending on which box
answered.

An unreachable satellite gets its own explicit amber "Satellite unreachable" state rather than
silently falling back to red "Inactive" — those mean genuinely different things to an admin (mail is
actually down vs. I currently can't tell), and collapsing them into one red dot would be worse than
useless during an actual incident. Both overlaid rows also grow an "on {server name}" label, and the
Start/Restart/Stop buttons are suppressed for them specifically — there's nothing local left to
control once mail has moved elsewhere.

The satellite dispatch is cached 20 seconds per active server — same TTL the dashboard's other
polled widgets already use — so neither the Service Status page nor the dashboard's live poll pays a
real network round trip on every load or tick.

## A drive-by fix, caught while writing this

Mail Settings' page already said, right under the "Active mail server" selector, that catch-alls and
mailing lists "don't route through the new server yet." That was true when it was written — before
Stage 3.4 shipped in the previous phase, which is exactly what fixed it. Left uncorrected it would
have told every admin who reads that page something false about a feature that had already been
fixed one phase earlier. Reworded to correctly name the one remaining gap: plain forwarders, still
noted directly in `MailAuthMapService`'s own doc comment as the explicit next target.

## Verified

Full local suite (1657 tests) and `pint --test` green. Live, for real: cut v0.16.37, upgraded
`mail.dev.knj.network` via its own Panel Updates page, then dispatched a real
`mail-service-status-report` from Main and confirmed the returned `postfix`/`dovecot` active state
matched `systemctl is-active postfix dovecot` run directly on the satellite over SSH — exact match.

## Stage 3, done

Everything laid out at the end of Phase 108 is now shipped: webmail routing, every mechanical
service conversion (Tier A), the two new provisioning actions for mail queue/log (Tier B), the two
new auth-map services for catch-alls/mailing lists (Tier C), and this Service Status split. What's
left for Mail Only isn't Stage 3 scope: plain forwarders' own auth-map push, DKIM, statistics, and
cross-server admin SSO (Stage 4) all remain open, tracked separately. Next up is the full end-to-end
verification pass across everything this stage built, together, against the real linked satellite.
