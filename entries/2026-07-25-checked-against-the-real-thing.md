# Phase 13 - Checked Against the Real Thing

Yesterday's Services settings pass covered the basics. This time, rather than continuing to work
from memory of what WHM and cPanel offer, we pulled up live demo instances of both and went
through them screen by screen to see what a real panel actually exposes — and to make sure
nothing obvious was being missed.

## What came out of it

**Nginx Settings** gained the security headers real hosting panels set by default —
`X-Frame-Options`/`X-Content-Type-Options` (on by default) and an optional HSTS toggle (off by
default — turning it on before every hosted site reliably serves HTTPS is actively harmful,
browsers refuse to fall back to plain HTTP for a year afterward) — plus a directory listing
toggle and basic per-IP rate limiting.

**PHP Settings** gained `max_input_vars` — the single most common real-world PHP limit people
actually hit (large WordPress admin screens, page builders) — along with `allow_url_fopen`,
session lifetime, and `disable_functions`, the actual security control shared hosts use to block
things like `exec`/`shell_exec` server-wide.

**Mail Settings** gained a per-client connection limit and an RBL (real-time blocklist) toggle
against Spamhaus.

**DNS Settings** gained a nameserver hostname override, so new zones can point at a branded
hostname (`ns1.yourcompany.com`) instead of the raw server hostname — matching a genuine WHM
concept, even though it turned out WHM's own page with that exact name does something slightly
different (picks which nameserver *software* to run, not the hostname). Good opportunity to
double-check BIND was still the right call: WHM's own listed advantages for it — manually
editable config, extremely configurable, tolerant of zone file mistakes — are exactly why it was
picked here in the first place.

## What we found and deliberately didn't build

The demo walkthrough surfaced several genuinely large features this panel doesn't have yet:
spam/malware scanning, full per-domain DKIM/SPF signing, hosting more than one domain per
account, any kind of traffic or log visibility for account owners, scheduled tasks, a general
app installer beyond WordPress, and account-level API tokens. Each of those is comparable in
size to a full milestone on its own — worth being upfront about rather than quietly skipping.

## Also: proved the new auto-deploy actually works

This whole change shipped without anyone pasting a deploy command — pushed to git, and the
[automatic deploy timer set up alongside yesterday's audit work](2026-07-25-closing-the-audit-follow-ups.md)
picked it up and went live within the same 5-minute cycle, on its own.

Multi-PHP version support — installing several PHP versions side by side, letting each account
pick its own, and isolating each account's own PHP settings from every other account's — is next.
