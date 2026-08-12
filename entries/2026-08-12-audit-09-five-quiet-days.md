# Phase 71 - Five Quiet Days, Checked Properly Instead of Assumed

First real break from this project — five days with nobody watching, after a run of long sessions
that earned it. Coming back, the ask wasn't "carry on where we left off," it was "prove nothing
happened while we were away" — every repo, every server, every log, checked properly rather than
just glanced at and trusted.

Also worth writing down for the record: Lemon Squeezy accepted the product during the break. The
video they asked for, done; the account's live. Whenever the release actually happens is a separate
decision, but the thing that was blocking it isn't blocking it anymore.

## What "checked properly" actually meant

Different shape from every audit before it, because there was no new code — the panel repo's
working tree came back byte-identical to what audit #08 deployed five days ago. So this pass wasn't
a code review; it was an intrusion-detection sweep, and the discipline was the same either way:
verify, don't assume.

Every accepted SSH login across all three live servers, for the entire five-day window, traced back
to exactly one IP — the operator's own. Not "looks about right" — the actual unique set, computed
directly, came back as a list of one. The panel's own login-brute-force jail (separate from SSH's)
showed zero attempts in five days, successful or otherwise. Firewall rules re-checked against what
they actually enforce, not just what the config file says: `knj-licence-server`'s internal API port
really is scoped to only the web server's IP, its backoffice really is bound to the Tailscale
interface only — confirmed by trying to reach both and getting exactly the outcome the design
promises.

Two things came up along the way that could have been findings and weren't, once actually chased
down instead of taken at face value:

A grep for "login" across `panel-dev`'s Laravel log surfaced a two-factor email OTP — worth
checking, since an unexpected login-code email during an unattended window is exactly the kind of
thing that matters. Its real timestamp turned out to be 2026-07-31, a week before the window even
started; the log file just accumulates everything, and an ungrounded text search doesn't know that.
Redone properly — grep scoped to the actual date range — and the real content of the window was
two lines total, one of them a single database connection error at 06:56:13 on the 11th. Traced
against `journalctl` for `mariadb.service` and cross-referenced with `unattended-upgrades.log`
directly: a routine systemd security patch restarted MariaDB as a side effect, and the queue
scheduler's own cache check landed in the one-second gap while it came back up. Not a fault at
all — genuine confirmation that automatic security updates are actually installing themselves
unattended, which is one of the panel's own Security Scan checks.

The second: `knj-licence-server`'s Tailscale-only backoffice answered a plain `curl` from this same
session with a real TLS handshake and a 302, not the connection failure the access control should
produce for an outside request. Worth stopping and tracing before writing it up as a regression —
`tailscale status` on this environment showed it's enrolled on the same tailnet the licence server
is, as `keith-crawlere10`. This is the operator's own trusted device reaching a Tailscale-restricted
service exactly as designed; the earlier audit's own "unreachable" result for the identical check
was the one that needed an asterisk, not this one. Same lesson underneath both: a surprising result
is a reason to look closer, not a reason to report it as-is in either direction.

## What's actually new

One real, if minor, gap: `knj-licence-server`'s Caddy instance has no HTTP access logging
configured anywhere — only its own TLS renewal housekeeping reaches its log. Database-level
querying (customer count, license count, the one admin account's last-modified timestamp, all
unchanged) was enough to establish nothing happened this time, but a real incident would want
request-level logs — status codes, paths, source IPs — not a reconstruction from what didn't
change in the database. Logged as an outstanding item, not fixed this pass; nothing pointed at it
being needed today, but it's a real hole in what could be reviewed if it ever were.

Everything else: clean. No unauthorized access, anywhere, on any protocol, for five full days. Full
report in `docs/security-audits/2026-08-12-audit-09.md`.
