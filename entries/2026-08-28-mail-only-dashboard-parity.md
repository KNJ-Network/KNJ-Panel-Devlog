# Phase 138 - The Dashboard Mail Only Never Got

Linking up a real production stack today — Mail Only and two DNS Only servers alongside the Main
install — surfaced something that had been quietly missing since Mail Only first shipped: it never
got the dedicated dashboard DNS Only got at the same stage. Log into a freshly installed Mail Only
box and it fell straight through to the generic Main dashboard — Accounts, Packages, Resellers cards,
none of which mean anything on a box with no Sites at all, plus a red "Your licence is not currently
valid" banner that shouldn't have been there either. Mail Only is licence-exempt, same as DNS Only,
and the exemption logic itself was already correct in every place that mattered — the banner slipped
through because it's embedded directly in the generic dashboard view with no role check of its own.
DNS Only only ever avoided it as a side effect of having its own separate view; Mail Only, never
having gotten one, inherited the bug by default.

Built the real thing this time, not a re-skin. A Mail Only box has no local database at all for
mailboxes — Main owns every `Mailbox`/`Site`/`Account` row and only ever pushes flat Postfix/Dovecot
config down to the satellite, the same "Main is the only source of truth" design DNS Only uses for
zones, just implemented as flat files instead of a database table because that's what Postfix and
Dovecot actually read. So the dashboard's four cards are all genuinely local, not proxied: Total
Mailboxes (a new provisioning action counting lines in `/etc/dovecot/mail-users`, the exact auth file
Dovecot itself authenticates against), Mail Queue depth (`postqueue -j`), Recent Deliveries
(`/var/log/mail.log`), and a Service Status panel filtered to the services this role actually
installs — Postfix and Dovecot instead of BIND9 and vsftpd.

Also fixed the same gap in the first-run setup wizard: DNS Only gets an inline "Link to Your Main
Server" step baked right into onboarding; Mail Only never did, silently falling back to a separate
trip to the Server Link page after finishing setup. Widened the same step to cover both roles, and
exposed the Mail Queue and Mail Delivery Reports pages in the Mail Only nav while at it — genuinely
diagnostic tools reading this box's own local state, not "mail configuration authored on Main," so
consistent with why Mail Only hides configuration pages in the first place.

Full local suite (2480 tests) and `pint --test` green. Deployed to `panel.dev.knj.network` and
confirmed live before cutting the release — nothing broken on the Main dashboard, the new Mail Only
branch renders correctly under test.
