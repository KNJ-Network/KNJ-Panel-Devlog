# Phase 118 - webmail.domain and domain.com/webmail

The last item picked up that day: closing a gap deferred back on 2026-07-25. The roadmap and the zone
service's own notes listed `webmail.<domain>` as a service-discovery record "deliberately not done",
on the reasoning that publishing DNS for a service nothing answers on is worse than not publishing it.
That was correct then, because KNJ Webmail did not exist yet. It has since shipped, and nobody went
back to close the loop. The DNS-only comparison audit surfaced the stale deferral.

## What comparable panels do, and what this ships

Comparable panels give every hosting customer two ways into webmail: `webmail.domain.com` and
`domain.com/webmail`. Both now work here.

**The DNS record**: a `webmail` record with the same toggle as `mail`, defaulting on, added to new
zones and to the self-service "Restore Default Records" action.

**`domain.com/webmail`**: every customer vhost already includes one shared snippet driven by
Controller, Nginx Settings. A redirect added there is live on every existing domain immediately, and
on every domain created afterward. It is a redirect rather than a proxy because the target already
has valid HTTPS on the panel's own hostname. It is omitted until a panel hostname exists.

**`webmail.domain.com`**: no customer vhost matches this name, so a catch-all vhost handles it,
written by the same action that writes the panel's own vhost, so the redirect target cannot go stale.
It is HTTP-only on purpose: a customer's certificate only covers that exact domain, so terminating
HTTPS here would present the wrong certificate. Customer vhosts with an exact name match are never
intercepted. It is limited to Main, since the satellite roles host no customer domains.

**Kept in sync on hostname change**: issuing the panel certificate now also re-applies the Nginx
settings, so existing redirects update immediately.

## Verified

Full local suite (1,708 tests) and the code-style check green, with new tests for the zone service,
templates, Nginx settings and server setup.

## Two more bugs found live, fixed same day

Re-issuing the dev server's panel certificate surfaced two pre-existing bugs in the certificate
action. First, the certificate tool exits non-zero for the benign "not yet due for renewal" case,
which aborted the action before the vhost rewrite; the output is now checked for that message.
Second, a lookup for the install's role that found nothing aborted under strict error handling before
its default could apply, on an older install that lacked a role entry; the empty result is now
tolerated. Shipped as v0.16.46 and v0.16.47.

## End-to-end, on the live dev server

With both fixes live, the generated configuration contained the redirect and the catch-all, and
against a test domain the `/webmail` path and the `webmail.` name both redirected to the webmail
login page, which loaded successfully. The test zone's missing record was backfilled through "Restore
Default Records" and resolved.
