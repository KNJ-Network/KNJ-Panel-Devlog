# Phase 154 - The Socket That Was Never Uncommented

Testing a genuine single-server stack turned up a fix for one Roundcube bug and one Dovecot plugin
bug this week. Sending mail from that same box worked fine both times. That made the next report —
outbound send failing on the real 4-server production stack, with both Roundcube and KNJ Webmail
throwing a plain "authentication failed" — look client-specific at first. It wasn't.

IMAP against the exact same mailbox, on the exact same box, worked without complaint. Only SMTP
submission failed, and Postfix's own `mail.log` had nothing to say about it — not even a rejected-auth
line, which Postfix normally logs even for a wrong password. A `postconf`/`ufw`/certificate sweep came
back clean: SASL was enabled in `main.cf`, the submission block in `master.cf` was correctly configured,
port 587 was open and listening, and the single TLS certificate on that box was shared correctly across
every service using it. None of it explained the silence.

The silence was the clue. Reading `mail.log` directly (not through any dashboard) surfaced the real
error, logged the moment anything tried to authenticate:

    postfix/smtpd: warning: SASL: Connect to Dovecot auth socket 'private/auth' failed: No such file or directory
    postfix/smtpd: fatal: no SASL authentication mechanisms
    postfix/master: warning: /usr/lib/postfix/sbin/smtpd: bad command startup -- throttling

Postfix asks Dovecot to check credentials over a Unix socket at `/var/spool/postfix/private/auth`.
That socket didn't exist. Every `smtpd` worker handling an AUTH attempt crashed outright the instant it
tried to reach it, and Postfix's own throttling of repeated crash-starts is exactly why some connection
attempts never produced a log line at all — the worker never got that far.

Dovecot's stock `10-master.conf` ships this listener commented out by default:

    # Postfix smtp-auth
    #unix_listener /var/spool/postfix/private/auth {
    #  mode = 0666
    #}

`setup-mail-server.sh` already uncomments this correctly — confirmed, since a fresh install (the
single-server stack from earlier this week) has it active with explicit `user = postfix` / `group =
postfix` ownership. The gap was never in that patch. It's in when it runs: `setup-mail-server.sh` is
only ever invoked from inside `issue-panel-cert`'s marker-gated "run the full mail+DNS setup once per
domain" block. Once that marker matches a box's domain, the script simply never runs again — no matter
how many times an admin re-saves Server Setup afterward. A box that slipped through with this listener
still commented out (whatever the original cause) had no path back to correct, short of a full
reprovision or someone reading this exact log line by hand.

Moved the same idempotent check — matched on the stable `service auth {` opening line, not comment
text that varies between Dovecot releases — into `issue-panel-cert`'s *unconditional* self-repair
section, right alongside the existing Roundcube `des_key`/logs-ownership/FPM-logging self-heals from
earlier this week. Same shape, same reasoning: a file read and one idempotent patch is cheap next to
the full mail+DNS install the marker exists to avoid re-running, so it's safe to check on every single
hostname/cert confirm. Restarts Dovecot only when the listener was actually missing.

Tested (2506/2506).
