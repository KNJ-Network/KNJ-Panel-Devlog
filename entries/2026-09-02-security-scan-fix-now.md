# Phase 173 - Not Every Warning Deserves the Same Button

Security Scan has run six checks against the server since it shipped back in Phase-whatever-that-
was-in-July — firewall, SSH, brute-force protection, unexpected listeners, database exposure,
automatic updates — and told the admin exactly what was wrong with each one. What it never did was
fix any of it. Every recommendation ended the same way: read the advice, go find the right page
yourself, do it by hand. The obvious next step was a "Fix Now" button. The less obvious part was
that six checks don't deserve the same button.

## Three kinds of fix, not one

The instinct with a feature like this is to wire up "Fix Now" as a single pattern and stamp it
across every failing check. That instinct doesn't survive contact with what these six checks
actually are. Restarting fail2ban is safe, synchronous, and has no real failure mode worth worrying
about. Flipping a firewall to default-deny can lock an admin out of their own server if it goes
wrong. Fixing "MariaDB is reachable from outside the server" isn't really this page's problem at
all — there's already a whole feature, Access Hosts, that owns that setting and rebuilds it from
scratch on every change; adding a second writer here wouldn't fix the exposure, it'd create two
features quietly fighting over the same `bind-address` line. And SSH configuration has a failure
mode that isn't hypothetical: the admin reading this scan might be connected over the exact password
auth the recommendation is telling them to disable.

So instead of one button, three: **auto** for brute-force and automatic updates, both genuinely safe
to fire immediately; **link**, which points at whichever page actually owns the underlying setting
instead of duplicating it; and **instructions**, real copy-and-run steps, for anything where "just do
it" is the wrong answer no matter how confident the button looks.

## The one auto-fix that isn't unconditionally auto

Firewall is the interesting case, because it's genuinely safe *most* of the time and genuinely
dangerous the rest of the time, and the difference isn't something the button can tell just from
which state the check reported. Enabling default-deny is fine if every port the admin actually needs
already has an explicit allow rule sitting in front of it. It's a potential lockout if one doesn't.
The fix here doesn't ask the admin to know the difference — a new read-only check
(`reservedPortsFullyAllowed()`) walks the same reserved-port list the Firewall page's own general-
rule feature already protects (SSH, the panel's own admin/account ports, mail, DNS, FTP and its
passive range) and confirms every single one already has an allow rule, regardless of how or when it
got there. Only then does the button actually fire. If it can't confirm that, the check falls back
to instructions instead — go add the missing rule first, then re-run the scan — rather than either
silently refusing or, worse, doing it anyway and hoping.

The provisioning-script side doesn't just trust that the PHP-side check already did this. It
re-verifies the same reserved-port coverage itself before touching anything, because this is the one
action in the whole script whose failure mode is "the admin loses access to everything, including
the sudo access this very script depends on." Belt and suspenders, on the one button in this feature
where suspenders alone isn't good enough.

## Async without lying about it

Two of the three auto-fixes — brute-force and firewall — can report success and immediately re-run
the scan inline, because both are synchronous and self-verifying by the time the request returns.
The updates fix can't. Installing `unattended-upgrades` needs `systemctl enable`, and that hit a real
wall on panel-dev: php-fpm's own sandbox (`ProtectSystem=full`) blocks that specific syscall even
after the config file itself became writable. Rather than punch another hole in the sandbox for one
action, the fix moved off the request path entirely, onto the same queue worker that already runs
outside the sandbox for exactly this class of problem — Perl Modules and PHP PEAR Packages hit the
identical wall months ago and got the identical fix. The button now says plainly that this one runs
in the background, check back in a few seconds, instead of pretending an immediate re-scan would mean
anything before the job's even started.

## Verifying it against something actually broken

2673/2673 passing, including a full breakdown of each check's fix classification under every state
it can report. Then the part that only a real server can confirm: disabled ufw entirely on panel-dev
and hit the button, confirmed default-deny came back and the re-scan showed Pass; separately removed
one reserved port's allow rule first and confirmed the same button refused to fire, falling back to
instructions instead of a coin-flip; stopped fail2ban and confirmed the button brought it back;
disabled unattended-upgrades and confirmed the button installed and enabled it for real. SSH and the
panel's own admin/account ports stayed reachable through the firewall test specifically — checked,
not assumed, the same discipline as every other change this build has made to live network filtering.

Tested (2673/2673).
