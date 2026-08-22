# Phase 112 - Mail Only, fully closed out

Three loose ends left from Stages 3 and 4, all the same shape: a feature that worked correctly on
Main but was never wired through the provisioning runner, so it kept acting against Main even once
a Mail Only satellite took over.

## Plain forwarders

Catch-alls and mailing lists got their own satellite auth-map push in Stage 3; plain forwarders were
the one remaining feature without one. A new forwarder auth-map service follows the shape of its
two siblings: it does nothing unless a Mail Only server is active, otherwise it pushes the full
current forwarder set, and it is wired into forwarder create and delete and into the resync that
runs on server switch. Before this, a forwarder created while mail was on a satellite was stored on
Main but produced no configuration on the satellite.

## DKIM

DKIM was not a missing feature. Installation, signing setup and per-domain keys were already built
and installed on Mail Only satellites. The bug was small: key generate and remove called the
provisioning step directly instead of through the runner, so keys were always generated on Main.
The fix was to route them through the runner.

## Mail statistics

The Mail Statistics page read the mail log from Main's own filesystem, so it showed empty or stale
data once mail moved to a satellite. It now uses the same log-read action the delivery-log service
already uses.

## Verified

Full local suite (1678 tests) and the code-style check green. Live: cut v0.16.39 and upgraded the
satellite via its own "Update Now". A test forwarder created on Main appeared on the satellite and
resolved; a DKIM key generated from Main landed on the satellite while Main's own setup stayed
untouched; and the statistics read from Main matched the satellite's real log. All test resources
were cleaned up afterward.

This closes out every open item in the Mail Only work: linking, dispatcher, service migration,
cross-server sign-in, and this final cleanup.
