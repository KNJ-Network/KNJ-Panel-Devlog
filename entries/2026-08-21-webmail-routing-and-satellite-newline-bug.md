# Phase 108 - Mail Only Stage 3 Begins: Webmail Routing, and a Bug Three Levels Deep

Started the next slice of Mail Only — migrating the ~20 remaining mail-related account features to
be satellite-aware, starting with the highest-impact piece: KNJ Webmail itself. Shipped that fix,
then found something bigger while verifying it live.

## Webmail was still talking to the wrong box

KNJ Webmail, its Settings page, and the account-side Roundcube launch link all resolved their
IMAP/SMTP/webmail host from `Server::where('is_self', true)` — always Main, with zero awareness of
Mail Settings' own "active mail server" selector. Every other mail-touching service already
resolves "where does mail actually run" through the dispatcher built in Stage 2; these three call
sites just never got the memo. Once an admin links and activates a Mail Only satellite, real
mailbox data has already moved there via the existing dispatcher — but webmail kept connecting to
Main regardless.

Fixed with a small shared resolver (`MailServerHostnameResolver`) — the same "active server, or
fall back to this box's own hostname" lookup the dispatcher already does internally, just exposed
as a hostname for the handful of call sites that need one to connect to rather than an action to
dispatch. Always a hostname, never an IP — whichever box terminates IMAP/SMTP presents a TLS cert
issued for its own name.

## A bug three levels deep, found by actually trying to log in

Live-verifying that fix meant checking a real linked satellite's mailbox actually worked end to
end — and it didn't. The satellite's Dovecot auth map was completely empty, despite a genuine
mailbox existing with `mail_routing` correctly set to route there. The dispatch itself reported
success the whole time.

Tracing it took three layers, each only visible by testing the real thing rather than reading the
code:

1. **Reproduced with a raw curl POST** carrying the exact JSON payload the app sends — still empty,
   ruling out anything specific to how the app builds the request.
2. **Instrumented the deployed provisioning script directly** (a temporary debug line capturing the
   received temp file before cleanup) — the file arrived with the right content, 194 bytes, correct
   text... except missing exactly one trailing newline versus what was sent.
3. **That one missing newline was the whole bug**, compounding two separate things: Laravel's
   `TrimStrings` middleware (part of the default request-handling stack this endpoint otherwise
   sits in) silently strips leading/trailing whitespace from every string field, including a
   generated config blob that was never meant to be trimmed — a byte-exact payload isn't form input
   from a person. And a genuine, well-known bash gotcha: `while read line; do ...; done < file`
   silently skips the last line of a file with no trailing newline, because `read`'s own exit status
   on that final call is nonzero even though it captured the data. With a single mailbox, "the last
   line" was "the only line" — hence a completely empty file that still reported success, since a
   short-by-one-line map is still valid config as far as `doveconf -n` is concerned.

Fixed at the actual root (excluding this endpoint's payloads from the trimming middleware, the same
way it's already excluded from CSRF) and hardened defensively (every provisioning-script loop that
reads pushed multi-line content now uses the `|| [[ -n "$line" ]]` idiom already used elsewhere in
this same script, so a dropped trailing newline from any future transport quirk fails safe instead
of silently). A new regression test drives the actual HTTP middleware stack rather than faking the
process call — the class of bug that made this invisible to begin with.

This also almost certainly explains an anomaly flagged two sessions ago and left unresolved: a test
mailbox that "never appeared" in a satellite's regenerated auth map across several attempts, with
no error anywhere. It was never a per-mailbox exclusion — it was always whichever entry happened to
land last.

## Verified

Full local suite and `pint --test` green. Reverted the middleware fix locally and confirmed the new
regression test genuinely fails without it, then restored it — not just a test that happens to pass.
Live, for real: cut a release, pushed it to both the dev Main server and a real linked Mail Only
satellite via its own Panel Updates page, then round-tripped a real IMAP `LOGIN` against the
satellite and got back `OK ... Logged in` for a mailbox whose auth data only exists there.

## Next

The rest of the ~20 remaining mail services — mail filters, greylisting, relay config, spam
settings, mail queue, delivery logs, catch-alls, and mailing lists — still need the same
satellite-aware treatment, plus a Service Status page that currently has no idea mail might be
running somewhere else entirely.
