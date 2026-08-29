# Phase 153 - The Quota Plugin That Needed a Friend

Testing webmail on a genuine single-server stack — not the 4-server production topology this whole
session had been living in — turned up a second bug, unrelated to the `array_first()` fatal from the
last three releases. Roundcube would log in fine, then throw random-looking `Server Error: CAPABILITY:
Internal error occurred` or `Server Error: STATUS: Internal error occurred` banners, never the same
command twice.

Dovecot's own log had the real story, on every single login, in plain English:

    imap(info@test.knj.network): Error: Couldn't load required plugin
    /usr/lib/dovecot/modules/lib11_imap_quota_plugin.so: Plugin quota must be loaded also
    (you must set: mail_plugins=$mail_plugins quota)

The generated Dovecot config had this:

    protocol lmtp {
      mail_plugins = $mail_plugins sieve quota
    }
    protocol imap {
      mail_plugins = $mail_plugins imap_quota
    }

`mail_plugins` is scoped per protocol block — `lmtp`'s own `quota` doesn't leak into `imap`'s block,
they're independent lists. `imap_quota` (the plugin that actually reports quota over the IMAP
protocol) hard-requires the base `quota` plugin to already be loaded in the *same* block. It never
was. Every IMAP session on this box has been starting with a half-initialized plugin stack since
whenever this config generator was first written — this just happened to be the first time anyone
did a real end-to-end webmail login test against a plain single-server install instead of the linked
Mail Only topology this session otherwise lived in all week.

The fix is a one-line addition to the `imap` block:

    mail_plugins = $mail_plugins quota imap_quota

Fixed at the source — `issue-panel-cert`'s config-generation step — which runs on every hostname/cert
confirm, so an already-provisioned install self-heals the next time an admin touches Server Setup or
a cert renews, same shape as the two fixes before it this week. No separate migration path needed.

Found the same day as a second, much smaller bug while chasing this one: `roundcube-diagnostics` was
tailing `/var/lib/roundcube/logs/errors` — a path that has never existed. Roundcube's own file logger
writes to `errors.log`. Fixed the filename; nothing else about that action changed.

Tested (2506/2506).
