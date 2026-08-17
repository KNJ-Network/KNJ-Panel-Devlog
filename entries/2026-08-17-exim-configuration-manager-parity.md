# Phase 96 - Two Small Ones: User Manager and the Postfix Config Gap

Tonight's list after audit #11 wrapped: two remaining parity items, User Manager and Exim
Configuration Manager. Neither turned out to need what its name implies.

## User Manager was already done under a different name

cPanel's User Manager solves one specific problem: let someone else help manage an account without
handing over the whole login. That's exactly what Manage Team already does — scoped collaborator
accounts with area-level access — just built for how this panel actually authenticates rather than
cPanel's bundled email/FTP/WebDisk sub-user construct, which has no equivalent here since there's no
separate "cPanel access" toggle to bolt onto an email account. Same job, done, just under a name
that didn't match. Marked closed in the roadmap rather than building a second mechanism for a
problem the first one already covers.

## Exim Configuration Manager, translated to what this install actually runs

This one's real name mismatch is more fundamental: the panel runs Postfix, not Exim, by deliberate
architecture choice from early on. So "port cPanel's Exim Configuration Manager" was never
literally on the table — the honest version of this item is "does the Postfix-equivalent control
surface exist," and the existing Mail Settings page already covered the anti-abuse basics
(message size, recipient limits, rate limiting, RBL, blocked recipients) but stopped short of the
ACL-level controls a hosting admin actually reaches for.

Widened it with four more: HELO/EHLO hostname validation, sender-domain verification (reject mail
claiming to be from a domain with no valid DNS), a custom SMTP greeting banner, and incoming TLS
policy (opportunistic vs required). All four apply through the same `postconf -e` /
`postfix check` / backup-and-revert-on-failure path every other write to this page already uses —
no new mechanism, just more restriction chains flowing through the one that exists.

The one piece of actual audit-adjacent work: the free-text banner field is exactly the kind of thing
that broke a DNS TXT record earlier tonight if left unescaped. The settings file this action reads
is parsed one `key=value` pair per line — a banner value containing a literal newline would inject
an extra, unvalidated line of its own into that file. Closed with the same discipline as the earlier
fix: reject the character at the point it's entered (Laravel's own validator), and reject it again
independently in the provisioning script itself, rather than trusting the first check alone.

Deliberately still not a raw `main.cf` editor — Postfix has several hundred tunable parameters and
exposing all of them raw isn't what any "Tweak Settings"-style page in this panel does for any other
service. Curated surface, same as everywhere else.

## The regex the test suite couldn't have caught

Live-testing this against panel-dev's actual Postfix turned up a real bug the moment it ran for
real: a completely ordinary banner — a hostname and a couple of words separated by spaces — got
rejected outright as "contains a control character or newline." The validation regex was
`[\ -~]` — meant to read as "space through tilde," the standard printable-ASCII range. Inside a
bash bracket expression that's not what it means: backslash isn't an escape character there, so the
range actually ran from `\` (0x5C) to `~` (0x7E), silently excluding space and everything below
backslash — digits, uppercase letters, most punctuation. Any banner with a space in it, which is
most of them, was being rejected.

The feature test suite was structurally incapable of catching this: it mocks the `Process` facade
entirely, so the real bash never runs during `php artisan test`. Only executing the actual script
against a real shell surfaced it. Fixed with `[[:print:]]`, the portable POSIX character class for
exactly "printable, no control characters" — verified against both an ordinary value and one with
an embedded newline before redeploying, then re-verified for real against panel-dev's live Postfix:
all four new settings applied and read back correctly through `postconf`, then reset back to
defaults once confirmed.

## What's left

Full suite green (1419, six new tests for the four added controls plus the newline-rejection case),
`pint` clean, deployed and live-verified against panel-dev's real Postfix — including the fix for
the bug that same live-verification pass caught.

## Next

DNS-only server profile work starts tomorrow — bigger scope, deliberately not started tonight.
