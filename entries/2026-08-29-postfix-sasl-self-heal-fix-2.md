# Phase 155 - The Self-Heal That Healed Nothing

Shipped v0.17.6 an hour ago to fix a real, confirmed bug: Postfix on an already-provisioned Mail
Only satellite couldn't reach Dovecot's SASL auth socket, because Dovecot's stock config ships that
listener commented out and nothing had ever gone back to uncomment it on that particular box. The
fix moved the same idempotent patch `setup-mail-server.sh` already applies on a fresh install into
`issue-panel-cert`'s unconditional self-repair section, so a routine Server Setup hostname re-confirm
would pick it up.

Rolled v0.17.6 out to all four production servers, then went to actually trigger the fix on the
affected box by re-submitting its (unchanged) hostname on Server Setup. The panel showed its usual
"issuing a certificate — runs in the background" banner, the queue log confirmed `IssuePanelCertJob`
ran and finished cleanly in 3 seconds. Reading `/etc/dovecot/conf.d/10-master.conf` over SSH
afterward to confirm: still commented out. Exactly as broken as before.

The fix's own guard condition was the bug. It decided whether the socket already existed with:

    grep -q '/var/spool/postfix/private/auth' "$DOVECOT_MASTER_CONF"

Dovecot's stock file ships the listener like this:

    # Postfix smtp-auth
    #unix_listener /var/spool/postfix/private/auth {
    #  mode = 0666
    #}

That commented-out stub contains the exact same path string the guard was searching for. `grep -q`
doesn't care about the leading `#` — it matched the dead stub just as happily as a live listener,
the guard's `! grep -q ...` came back false, and the self-heal block never ran at all. A no-op,
silently, on precisely the box it existed to fix. The same flaw was sitting in `setup-mail-server.sh`
itself too — its Python check used `'/var/spool/postfix/private/auth' not in content`, the identical
substring trap, for both the LMTP and auth listeners.

Fixed both to require a match against an *active* listener line specifically — anchored at the start
of a line, allowing only leading whitespace before `unix_listener`, so a `#` prefix can never satisfy
it:

    ^[ \t]*unix_listener /var/spool/postfix/private/auth

Verified the corrected pattern directly against both the stock commented text and a properly patched
block before touching the real fix, rather than trusting it by inspection alone.

The lesson isn't new — it's the same one `setup-mail-server.sh`'s neighboring LMTP/auth patch already
learned once, about anchoring on stable structural markers instead of text that varies or, in this
case, text that's present either way. A "does this string exist in the file" check is never a safe
proxy for "is this feature active" when the file's own stock template already contains the string as
inert filler.

Tested (2506/2506).
