# Phase 108 - Mail Only Stage 3 Begins: Webmail Routing, and a Bug Three Levels Deep

Started the next slice of Mail Only: making the remaining mail-related account features
satellite-aware, starting with KNJ Webmail itself. Shipped that fix, then found something bigger
while verifying it live.

## Webmail was still talking to the wrong box

KNJ Webmail, its Settings page, and the account-side Roundcube launch link resolved their mail host
from the local server's own record, with no awareness of Mail Settings' "active mail server"
selector. Once an admin activated a Mail Only satellite, mailbox data had moved there but webmail
kept connecting to Main. A small shared resolver now returns the active server's hostname (always a
hostname, never an IP, since the TLS certificate is issued for the name).

## A bug three levels deep, found by actually trying to log in

Live verification showed that the satellite's mail auth data was empty even though a real mailbox
existed and the dispatch reported success. Tracing took three steps, each only visible by testing
the real thing: reproducing with a raw request ruled out the app, instrumenting the deployed
receiving side showed the payload arrived missing exactly one trailing newline, and that single
byte turned out to be the whole bug. Two things compounded: request-handling middleware silently
trimmed whitespace from a generated configuration payload that was never meant to be trimmed, and a
well-known shell gotcha caused the final line of a file with no trailing newline to be skipped. With
a single mailbox, the last line was the only line, so the result was an empty file that still
passed validation.

It was fixed at the root by exempting these payloads from trimming, and hardened so that scripts
reading pushed multi-line content handle a missing final newline. A new regression test drives the
real request-handling stack rather than faking the call.

This also explained an earlier anomaly where a test mailbox "never appeared" in a satellite's auth
data: it was never a per-mailbox exclusion, only whichever entry came last.

## Verified

Full local suite and code-style check green. We reverted the fix locally and confirmed the new test
fails without it. Live: cut a release, pushed it to the dev Main server and the linked Mail Only
satellite via its Panel Updates page, then completed a real IMAP login against the satellite for a
mailbox whose auth data exists only there.
