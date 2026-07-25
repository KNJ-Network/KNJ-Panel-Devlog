# Phase 13 - Checked Against the Real Thing

Yesterday's Services settings pass covered the basics. This time, rather than continuing to work
from memory of what a full-featured hosting panel needs, we went back through established hosting
panel conventions screen by screen to see what a real panel exposes — and to make sure nothing
obvious was being missed.

## What came out of it

**Nginx Settings** gained standard security headers (on by default) and an optional HSTS toggle
(off by default, since enabling it before every hosted site reliably serves HTTPS can lock
visitors out), plus a directory listing toggle and basic per-IP rate limiting.

**PHP Settings** gained `max_input_vars` — a limit people commonly hit with large WordPress admin
screens and page builders — along with `allow_url_fopen`, session lifetime, and
`disable_functions`.

**Mail Settings** gained a per-client connection limit and a blocklist toggle.

**DNS Settings** gained a nameserver hostname override, so new zones can point at a branded
hostname instead of the raw server hostname.

## What we found and deliberately didn't build yet

The walkthrough surfaced several larger features the panel doesn't have yet: spam/malware
scanning, full per-domain DKIM/SPF signing, more than one domain per account, traffic and log
visibility for account owners, scheduled tasks, and a general app installer beyond WordPress.
Each is comparable in size to a full milestone on its own, so they were noted rather than rushed.

## Also: automatic deploys work

This change shipped without anyone pasting a deploy command — it was pushed to git and picked up
by [the automatic deploy mechanism set up alongside yesterday's audit work](2026-07-25-closing-the-audit-follow-ups.md).
