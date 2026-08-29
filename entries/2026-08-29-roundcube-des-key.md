# Phase 142 - A Key That Was Never 24 Characters Long

Straight on from Phase 141: with the nginx role gate fixed and pushed, "Open Webmail" on the same
Mail Only satellite stopped 403'ing — and immediately started 500'ing instead. Nginx was now
routing to Roundcube correctly. Roundcube itself was falling over.

Nothing useful came out of the obvious places. Nginx's error log showed a clean upstream response,
just a 500 body. Roundcube's own log directory existed and looked right. An investigation agent
went looking for what could make Roundcube fail on literally every request, and came back pointing
at `setup-mail-server.sh`'s generation of the `des_key` — the value Roundcube uses to encrypt the
IMAP password it stores in the session, which has to be exactly 24 characters for its default
cipher. The script reads a fixed 64 raw bytes from `/dev/urandom`, filters down to alphanumeric
with `tr -dc 'A-Za-z0-9'`, then slices the result to `${DES_KEY_RAW:0:24}`. The comment sitting
right above it says 64 bytes "reliably yields well over 24 alphanumeric characters."

That comment is wrong, and not in a subtle way. Each raw byte only has a 62-in-256 chance — a bit
under a quarter — of landing in `[A-Za-z0-9]`. Sixty-four bytes filtered that way typically survive
down to 15 or 16 characters, not "well over 24." Didn't want to ship a fix on the strength of that
math alone, so before touching anything, ran the actual buggy generation logic standalone ten times
in a harmless local test. It never once reached 24 characters — it topped out somewhere around 7 to
22. Every single production install that had ever run this script got a `des_key` shorter than
Roundcube expects, silently. Roundcube doesn't refuse to start with a bad key — it throws trying to
encrypt or decrypt the stored IMAP password on every request, and that throw surfaces as a bare 500
with nothing diagnostic anywhere in reach.

Nothing anywhere means nothing, so we went looking for why even Roundcube's own internal log had
stayed empty through all of this. Found a second, independent bug sitting right next to the first:
`/var/lib/roundcube/logs` was chowned to `root:roundcube`, same as `/var/lib/roundcube/temp` next
to it, but only `/temp` had actually been `chmod 770`'d. `/logs` kept the package's default mode,
group-readable but not group-writable — so the `roundcube` pool user could never write there either,
regardless of the key bug. Two separate reasons the real failure was invisible, both hit at once.

The fix for the key: stop trusting one bounded read to be enough. Loop, actually counting survivors
after each `/dev/urandom` pull, and keep pulling more until there are genuinely 24 or more, then
slice to exactly 24. Ran that loop 8 times locally before trusting it — 8 out of 8 produced exactly
24 characters, a real result instead of an assumption about probability. The existing-key check
also got sharper: it used to just ask "does the file exist," which meant a server that already had
this bug would carry the broken key forever, since the file was present, just short. Now it checks
the length too, so the fix is self-healing the next time the script runs against an already-broken
install. The `/logs` permission line got folded into the same `chmod 770` as `/temp`.

That would have been enough for a fresh install, but not for the one server we actually needed
fixed. `setup-mail-server.sh` is marker-gated to run once per domain, and the live Mail Only
satellite's marker already matches — its own fix would never fire again without a real domain
change. No SSH access to that box either; this whole audit deliberately treats these servers like a
customer's, self-managed only. So `issue-panel-cert` — the action that already runs on every live
hostname/cert re-confirm, same one Phase 141 leaned on — picked up a small unconditional self-repair
step: on every call, for `main` and `mail_only` roles with a Roundcube config already in place,
check whether `/root/.roundcube_des_key` is missing or not exactly 24 bytes, and if so regenerate it
and `sed`-patch just the `des_key` line of `config.inc.php` in place. Cheap enough — a file read, a
few `/dev/urandom` pulls, one `sed` — that it's safe to run on every call instead of hiding behind
another marker. Tested the `sed` pattern separately against a realistic sample config line before
trusting it against the real file: exactly one match, no collateral changes to anything else in the
config.

Two independent causes, one investigation session, both invisible until the layer above them
(nginx) got fixed first. VERSION bumped to 0.16.94.

Tested (2506/2506) — no PHP changed, this is pure shell, but the project runs the full suite as a
checkpoint before every version bump regardless.
