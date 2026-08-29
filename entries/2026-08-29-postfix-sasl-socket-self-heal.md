# Phase 154 - A Mail Setting That Was Never Switched On

Testing a single-server install had turned up fixes for one Roundcube bug and one mail-server plugin bug
that week. The next report, outbound sending failing on a multi-server test install with a plain
"authentication failed" in both webmail clients, looked client-specific at first. It was not.

Reading the mail log directly surfaced the real error: the mail transfer agent could not reach the
mailbox server's authentication service, so it reported no sign-in methods available and began throttling
its own failing workers. That throttling explained why some attempts left no log line, and why a sweep of
configuration, firewall and certificate checks came back clean.

The mailbox software ships that connection switched off by default. Our setup already enables it on a
fresh install; the gap was in when the setup runs. It only ran once per domain, so a server that had
missed it had no way back to a correct state.

The same check, tied to a stable part of the configuration rather than to wording that changes between
releases, now runs whenever a server's hostname or certificate is confirmed. It is cheap next to the full
setup, safe to repeat, and restarts the mail service only when the setting was actually missing.

Tested (2506/2506).
