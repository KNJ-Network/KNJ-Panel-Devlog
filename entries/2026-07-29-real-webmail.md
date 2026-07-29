# Phase 20 - A Real Webmail Client

The last open item in Email: a proper in-browser mail client. Real cPanel and WHM don't build
their own webmail from scratch either — they bundle an existing, proven IMAP client and wire it to
the real mail server underneath. Same approach here: a real, well-established webmail application,
talking to the exact same mail server every other feature in this build already uses, not a
separate simulated one.

## What shipped

A shared webmail install, reachable from the panel's own address, logging in with the account's
real mailbox address and password — the same credentials used everywhere else. Its own database,
its own isolated process pool running under its own dedicated system user (not sharing an identity
with anything else on the box), and its own generated encryption key and database password,
following the same pattern already used for every other credential this panel manages internally.

## Three real bugs, found live

Three separate issues showed up only by actually testing this end to end against the real server,
not from documentation:

- A web-server configuration shortcut that's normally fine for serving a subdirectory of an
  application quietly breaks the standard mechanism PHP uses to figure out which file to actually
  execute — the practical effect was every request either 404ing or 403ing depending on exactly
  which one was requested, with nothing useful in any log to explain why.
- The webmail application's own config file needs to be readable by the specific system user its
  process runs as — an easy thing to get subtly wrong, and it fails *completely silently*: get it
  wrong and the application just quietly falls back to its own placeholder example database
  connection instead of raising any error at all, which took real tracing through the
  application's own startup code to actually catch.
- The mail server's connection settings need to point at this server's real hostname, not the
  generic local-loopback address — the TLS certificate involved is issued for the real hostname,
  and PHP's own secure-connection handling correctly refuses to connect somewhere the certificate
  doesn't match, just without a very clear error message explaining why.

None of these were exotic — each one's the kind of thing that's obvious in hindsight and invisible
until you actually try the real thing end to end, which is exactly why every feature in this build
gets tested that way rather than just checked into the database.

## Then proven against a real inbox, not just accepted by the mail server

A first pass confirmed the mail server itself accepted a message from webmail — necessary, but not
the same thing as proving it actually reaches someone's real inbox, since a throwaway domain that
isn't genuinely registered anywhere publicly resolvable can never pass authentication at a real
provider no matter how correctly the sending side behaves. So a real subdomain was properly
delegated to this server as its own authoritative nameserver, a real account created for it through
the actual interface, and a message sent from the real webmail login to an outside Gmail address —
confirmed landing in the inbox via the receiving mail server's own log entry, not just the sending
side's. Replying back was received correctly too, confirming the round trip both ways.

## Verified for real

A real mailbox was created through the panel's own account-creation code, logged into through the
webmail application's real login form — not a simulated session — and a real message was sent and
received through it, confirmed via a separate direct mailbox check. The install process itself was
then run twice more from a completely clean slate to confirm it's genuinely repeatable, not a
one-off that happened to work.

## What's still planned

A second, custom-built webmail client of our own, further down the road — the real IMAP protocol
edge cases and the security work around safely rendering arbitrary HTML email are worth doing
properly rather than rushed, so this is being sequenced deliberately rather than attempted
alongside everything else. The webmail shipped today stays available permanently either way; once
a custom client exists, the account panel will let each account owner choose which one they'd
rather use.
