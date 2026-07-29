# Phase 20 - A Real Webmail Client

The last open item in Email: a proper in-browser mail client. The established approach is bundling
an existing, proven webmail application rather than building one from scratch — same here: a
well-established webmail application talking to the same mail server every other feature in this
build uses.

## What shipped

A shared webmail install, reachable from the panel's own address, logging in with the account's
real mailbox address and password. It runs with its own database and its own isolated process
identity, following the same pattern used for every other credential the panel manages internally.

## Three real bugs, found live

Three issues showed up only by testing end to end against a development server, not from
documentation:

- A web-server configuration shortcut that's normally fine for subdirectories interfered with how
  PHP locates the file to run, producing inconsistent errors with nothing useful in any log.
- The webmail application's configuration needed specific file access, and when it was wrong the
  application silently fell back to placeholder settings instead of raising an error.
- The mail connection settings needed to use the server's real hostname, since the TLS
  certificate is issued for it and PHP correctly refuses a mismatch.

None of these were exotic — each is obvious in hindsight and invisible until you try the real thing
end to end, which is why every feature in this build is tested that way.

## Then proven against a real inbox

Confirming that the mail server accepted a message from webmail isn't the same as proving it
reaches a real inbox. So a test subdomain was delegated to the development server, an account was
created for it through the actual interface, and a message sent from webmail to an outside Gmail
address was confirmed delivered via the receiving server's log. A reply was received correctly too.

## Verified for real

A real mailbox was created through the panel, logged into through the webmail login, and a message
was sent and received through it. The install process was then run twice more from a clean slate to
confirm it's repeatable.
