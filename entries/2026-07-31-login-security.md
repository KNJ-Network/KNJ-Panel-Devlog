# Phase 38 - Locking the Front Door: Brute-Force Protection and 2FA

Second of the four v1.0 items, and the one this panel had the least excuse for not having yet:
fail2ban has protected SSH on every server since day one, but nothing protected the panel's own
login form — the only thing standing between "5 attempts" and "unlimited attempts" was Fortify's
own 60-second in-app throttle, invisible to the admin and trivially waited out.

**Brute-force lockout reuses the exact infrastructure already proven for SSH**, rather than
inventing a parallel app-level system. Every failed panel login now writes one line to a dedicated
log file; a new `knjpanel-auth` fail2ban jail watches it and bans the IP at the network level
(nftables — the same action the SSH jail already uses) after five failures in ten minutes. It
self-installs on every server through the same sync mechanism that already keeps the provisioning
script and systemd units current — no manual step, no SSH required, matching how this whole panel
has to manage itself once it ships to someone who doesn't have a terminal. Live-testing it turned
up something the unit tests never could: the jail silently inherited Ubuntu's `backend = systemd`
default, correct for sshd (which logs to the journal) but wrong for a jail watching a plain file —
it started fine, logged no error, and simply never saw a single line the app wrote. An explicit
`backend = auto` fixed it; confirmed after the fix by tailing fail2ban's own log and watching it
actually count a real failed login for the first time.

**Two-Factor Authentication turned out to be mostly already there.** Fortify has shipped 2FA
support the whole time — the database columns even already existed from an earlier, apparently
abandoned pass — just fully commented out. Wired it up properly: opt-in for every role (admin,
reseller, account owner), not required of anyone, on a new shared "My Profile" page reachable from
both the Controller and Account sidebars. QR code setup, a confirmation step before it actually
turns on, recovery codes, disable.

**Added a third fallback beyond what Fortify ships**: an emailed one-time code, for the case where
someone's lost their authenticator *and* never saved their recovery codes. Fortify only ever
checks an authenticator code or a recovery code, so this meant a fully separate controller
alongside its own — same session gate, same login-and-redirect shape, so all three paths behave
identically to whichever one succeeds.

Every path got tested against real generated codes, not just request shapes — enable, confirm,
disable, a full login-time challenge with a valid code, a recovery code that works once and is
correctly refused the second time, an emailed code round-tripped through the actual mail log. That
caught two real bugs in one afternoon: Google2FA's own replay protection rejecting a second code
generated from the same 30-second window as one just confirmed (only shows up if you actually try
to use two codes close together, which a unit test naturally does and a casual click-through
might not), and an invalid `<form>` nested inside another `<form>` that made the "email me a code"
button silently submit the wrong endpoint — something no amount of route-level testing could have
caught, since there's no DOM to misbehave in a raw HTTP request. Both fixed, both reproduced and
confirmed fixed live in a real browser before shipping.
