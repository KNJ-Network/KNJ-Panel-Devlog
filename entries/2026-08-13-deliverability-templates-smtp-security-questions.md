# Phase 75 - Working Down the Gap List

Back to the open items from the earlier click-through comparison: a client-side deliverability
checker, a stale tracking note, a web page template editor, an outbound mail-abuse protection, and
an unfamiliar-login challenge.

## Email deliverability, for domain owners

The admin side already had a per-account deliverability check (SPF, DKIM, DMARC, PTR). The client
side didn't, so a domain owner had to ask support to see their own mail authentication status.
This reuses the same DNS-lookup service and adds a Repair button per domain, wired to the DNS zone
service that already knows how to write correct authentication records. No new privileged action
was needed.

## One line that was quietly wrong

An internal tracking note claimed the mail configuration manager was missing a couple of tabs.
Both had since shipped. The record was fixed.

## A web page template editor

Suspended and default placeholder pages were fixed HTML baked into the provisioning script. Now an
admin can edit both from the panel with a live preview and a "restore built-in default" button.
Getting the new storage location working took one extra step in provisioning, so that a fresh
install creates what the feature needs ahead of time, matching the rule that no operator should
have to SSH in to set something up.

## Limiting outbound mail abuse

A compromised hosting account could in principle try to send spam directly rather than through the
server's mail system, bypassing its queue, rate limits, and signing. A new toggle restricts
outbound mail to the mail server's own delivery process. Genuine mail from the server's mailboxes
is unaffected, which was verified live, and the change is syntax-checked and automatically
reverted if anything looks wrong.

## An extra check when a login doesn't look like you

An unfamiliar-IP login can now be interrupted with a security question, on top of whatever else is
required. Each admin or reseller sets their own question and answer, and enabling it is a separate
org-wide switch, off by default. Someone with nothing configured is never challenged. A correct
answer doesn't finish the login by itself: it hands off into whichever check the account already
had configured, so two-factor protection isn't bypassed.
