# Phase 110 - Mail Only Stage 3 Wraps Up: Service Status Stops Lying About Where Mail Runs

Closed out the last piece of Stage 3: Service Status, both the admin page and the dashboard
widget, did not know mail might be running on a linked satellite. It only checked local services,
so once an admin activated a Mail Only server, both kept reporting Main's own now-irrelevant mail
state.

## The overlay, not a rewrite

The core service-status code did not need to change. The satellite-awareness lives one layer up: an
overlay replaces just the mail rows with the active satellite's real state, using the same "is this
unit active" check the local service runs, so "active" never means two things.

An unreachable satellite gets an explicit amber "Satellite unreachable" state rather than falling
back to red "Inactive", since "mail is down" and "I can't currently tell" mean different things to
an admin. Overlaid rows show an "on {server name}" label, and local Start/Restart/Stop buttons are
suppressed for them. The satellite query is briefly cached so page loads and dashboard polls do
not pay a network round trip each time.

## A drive-by fix

Mail Settings still said catch-alls and mailing lists "don't route through the new server yet",
which had been fixed a phase earlier. The text now names the one remaining gap, plain forwarders.

## Verified

Full local suite (1657 tests) and the code-style check green. Live: cut v0.16.37, upgraded the
linked satellite via its own Panel Updates page, then dispatched a real status report from Main and
confirmed it matched a direct check on the satellite.

## Stage 3, done

Everything planned for Stage 3 is shipped: webmail routing, the mechanical service conversions
(Tier A), the new mail queue/log actions (Tier B), the auth-map services for catch-alls and
mailing lists (Tier C), and this Service Status split.
