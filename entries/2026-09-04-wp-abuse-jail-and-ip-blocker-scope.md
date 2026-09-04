# Phase 200 - The Alarm That Only Covered Its Own Front Door

An incident found by accident — a Resource Monitor screenshot showing three php-fpm workers pegged
at a steady 30% — turned into a real gap in two different security features at once, both traced
back to the same root cause: they were both built to protect this panel, not the sites it hosts.

## What the screenshot was actually showing

The CPU wasn't a mystery for long. A single IP had sent 783 requests to `xmlrpc.php` on one
customer's site in about nine minutes — the textbook WordPress abuse pattern: discover the endpoint,
then hammer it. Not a repeat of the cryptominer that had compromised this same account earlier in
the week — a completely different, opportunistic attacker finding a live target and taking a swing
at it. The immediate fix was blunt and manual: block the IP with `ufw`, then discover that an
already-open connection doesn't care about a new firewall rule and has to be killed directly with
`ss -K` before the flood actually stops.

## Two alarms, both wired to the wrong building

Looking for the real fix surfaced two features that had quietly grown the same blind spot, for the
same underlying reason.

fail2ban already had two jails running on every server — one for SSH, one for this panel's own
login page. Neither had ever looked at a customer's site. The obvious next move, reaching for
fail2ban's own bundled `nginx-botsearch` filter, turned out to be the wrong tool for this specific
job: it matches requests for paths that don't exist, a 404-probe pattern. `xmlrpc.php` is a real
file. This wasn't a bot looking for something that isn't there — it was abuse of something that is.
That needed a filter built for this exact shape: repeated POST requests to `xmlrpc.php` or
`wp-login.php`, regardless of what nginx returned for them. Verified against a real captured access
log line with `fail2ban-regex` before it ever touched a live jail file, then verified again by
actually triggering it — appending a synthetic flood to a real site's access log and watching
`fail2ban-client` report a genuine ban.

The IP Blocker had a worse problem: the panel's own IP-blocking feature would not have stopped this
specific incident even if someone had reached for it, because it only ever wrote its deny list into
an account's *primary* domain. The site actually under attack was a subdomain. The copy on the
IP Blocker page said "block a specific visitor from reaching any of your sites" — a claim that was
simply false for anything but the primary one.

## Fixing "which sites" without losing "which address"

The fix needed a real scope, not a blanket switch. A nullable `site_id` on the block itself — null
still means every site, exactly the behavior this feature always had; a set value means one
specific domain and nothing else. The part worth being careful about was the database constraint
protecting against duplicate blocks: the obvious move, widening the existing unique index to include
`site_id`, quietly breaks in MySQL and MariaDB, where every `NULL` in a unique index counts as
distinct from every other `NULL`. Two "block this address everywhere" rows for the same address
would have silently both been allowed. The actual fix was simpler than the one that looked obvious:
leave the original constraint alone entirely. An address already blocked account-wide and a narrower
block for the same address on one site aren't two different states worth allowing side by side —
one row per address per account, whichever scope it happens to be set to, was enough all along.

Rebuilding the actual nginx config now means walking every site the account owns and writing each
one its own denylist, computed independently — "does this block apply to this site" instead of "here's
the one shared list." A site with nothing scoped to it still gets rewritten with an empty list, which
is what actually clears an old block once it's removed rather than leaving a stale deny rule sitting
in a vhost forever.

## The door nobody had put a handle on

There was a second, quieter gap behind the first one: the IP Blocker had always been an
account-owner-only feature. No admin-side page existed at all — which is the actual reason the real
incident that night had to be resolved over SSH instead of through the panel meant to make SSH
unnecessary. A new Controller-side page reuses the exact same service and the exact same rows the
owner's own page already uses — a block added by an admin shows up on the owner's page immediately,
and vice versa, because it was never two features, just one that had only ever been reachable from
one side of the panel.

Tested (2852/2852), pint clean, verified live on panel-dev end to end: the jail actually bans a real
synthetic flood, and a block scoped to one site writes only that site's own snippet while every
other site on the same account is left alone.
