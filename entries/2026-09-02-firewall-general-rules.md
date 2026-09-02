# Phase 171 - A Firewall Page That Actually Does Something

The Firewall page under Controller → Security had been sitting there since early in the build as
basically a placeholder: `ufw status verbose`'s raw text dumped into a `<pre>` block, fail2ban's
status the same way, and the only real actions on the whole page were the two "Currently Banned
IPs" unban buttons. No way to add a rule, no way to remove one, nothing structured. The operator
called it out directly — it "looks shit and is not editable" — and asked for a real, structured,
editable rule table matching the rest of the panel. Given what this page actually controls, getting
it wrong was a real way to lock an admin out of their own server over SSH, so this one went through
the same plan-first discipline as the temp-password feature two phases ago: a full research pass
over the real code, live verification of actual `ufw` output on panel-dev, and a second pass that
deliberately looked for ways the first draft of the design could go wrong before any code got
written.

## Not the same thing as Access Control

This codebase already has a "Firewall" of sorts — the Access Control page, IP allow/deny for five
fixed services (Panel Login, SSH, FTP, Mail, DNS). It was tempting to just extend that. Wrong move:
Access Control's `FirewallRule` table only has `(service, ip_or_cidr)` — no port, no protocol, no
allow/deny — and its whole sync model is "rebuild this one service's entire allowlist from the DB
every time," which only makes sense because every row for a service maps onto that service's own
fixed, hardcoded port list. A general rule isn't "restrict access to a known service," it's an
arbitrary port/protocol/action/source tuple with no such fixed list to rebuild against. Trying to
bend one feature's data model to also be the other's would have meant two different mutation
patterns fighting for ownership of the same table. Easier and safer to keep them genuinely separate:
new `firewall_general_rules` table, new `FirewallGeneralRule` model, Access Control untouched.

## Knowing which rule is whose, from a comment

`ufw` doesn't hand back a stable ID for anything — `ufw status numbered`'s numbers reshuffle every
time a rule is added or removed anywhere in the ruleset, the classic footgun. But this codebase
already had the answer to that sitting in front of it: `firewall-rule-write` (Access Control's own
provisioning-script action) already tags every rule it writes with a UFW comment
(`knjpanel-fw-{service}`), and a completely separate action for MariaDB remote-access rules does the
same thing with its own tag. So general rules get their own tag,
`knjpanel-fw-general:{the row's own DB id}`, and a new `UfwStatusParser` class turns
`ufw status numbered` text into structured rows that get classified by that tag into exactly one of
three buckets: **General** (this feature's own rule, editable), **Access Control** (someone else's
tagged rule, shown read-only with a link to where it's actually managed), or **System** (no
`knjpanel-fw-*` tag at all — bootstrap-server.sh's original baseline, or something an admin added by
hand — always read-only). That third bucket is the actual SSH-lockout guard: nothing this page shows
as deletable was ever put there by bootstrap-server.sh.

One thing worth writing down since it took an actual round-trip to the real box to catch: a naive
tag match (`grep -F "knjpanel-fw-general:5"`) also matches a rule tagged `:50`. Access Control gets
away with a plain substring match because its five service names are a closed set where none is a
prefix of another — that safety property doesn't hold once the tag is a sequential number. Both the
PHP-side classifier and the script's own delete-by-tag loop anchor the match at the end of the string
instead.

## Two features, one port, never at the same time

The other real risk with a second "add arbitrary firewall rules" feature living next to Access
Control is the two of them fighting over the same port — writing competing tagged rules whose `ufw`
ordering interaction nobody had actually reasoned through. Rather than try to reason through it,
general rules are blocked outright from touching any port bootstrap-server.sh's baseline setup or
Access Control already manages — both allow *and* deny, not just deny, since even an allow on an
already-managed port creates the same unanalyzed-interaction risk. That closes off the self-lockout
case too: an admin can't accidentally deny 2087 (the panel's own admin port) through this page, full
stop, checked before the request ever reaches the provisioning script.

For rules that *do* land on their own genuinely free port, there's still an ordering question:
what happens when an admin allows one source and later denies another, on the same port? `ufw`
evaluates top to bottom, first match wins, so insertion order matters. The rule here: allows always
insert at the very top, denies always append at the very bottom. Since general rules can never share
a port with anything baseline/Access-Control owns, the only rules that can ever share a port are
other general rules, and "every allow above every deny" is then a safe invariant no matter what order
an admin happens to add or remove things in.

## Read-only status, routed through the script anyway

The existing `ufwStatus()` (`ufw status verbose`, the raw text still shown in a collapsed "Advanced"
section) works via a direct sudoers grant — `sudo /usr/sbin/ufw status verbose`, nothing else. The
new structured read needed `ufw status numbered` instead, and the obvious move looked like "just add
one more direct sudoers line the same way." Wrong, for a reason specific to how this panel deploys
itself: `bootstrap-server.sh` writes sudoers once, at initial provisioning, and `knjpanel-upgrade`
never touches `/etc/sudoers.d/*` on an upgrade — only the provisioning script and systemd units get
resynced. A new direct grant would work on a brand-new install and never reach any server already
running today, panel-dev included, without someone manually patching sudoers by hand. Routed the
read through a new `firewall-status` action in the provisioning script instead — no new sudoers line
needed at all, since the script's own grant is unconditional and already resyncs correctly on every
upgrade.

## Verifying it for real

2638/2638 passing locally (28 new tests: a pure-parser suite built against real `ufw status
numbered` output captured from panel-dev rather than hand-typed approximations, an ownership-
classification suite with an explicit id-5-vs-id-50 regression test, and a feature suite covering
add/remove, reserved-port rejection — including a partial-range-overlap case, not just an exact
match — and a rollback-preserves-the-original-id regression for the failure path). Then the real
thing, live on panel-dev: added a rule on a genuinely unused test port through the actual UI,
confirmed the exact tagged, correctly-ordered rule appeared in a real `sudo ufw status numbered`;
tried to add a deny rule on 2087 (the panel's own admin port) and confirmed it was rejected client-
side before the request ever reached the provisioning script, with the real baseline rule left
completely untouched; removed the test rule through the UI and confirmed it was genuinely gone from
the live ruleset, not just from the panel's own display. SSH and the panel's own admin/account ports
stayed reachable throughout — the one feature this session where that needed an explicit check
rather than an assumption.

Tested (2638/2638).
