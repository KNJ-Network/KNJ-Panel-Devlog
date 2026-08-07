# Phase 70 - A Roadmap That Lied by Omission, and Two Bugs Set-e Was Hiding

Same instruction as before audit #07: full stop, check everything, don't move on until it's clean.
This time the ask went wider than usual — not just an audit of what shipped since the last one, but
a full roadmap accuracy sweep first (every unmarked item cross-checked against the real code, every
public roadmap row checked for a missing note), then the audit itself, then a live GitHub cross-check
across every repo in the fleet. The user was explicit about why: every audit so far has found
something, and the next time one doesn't, it should be because this one was thorough enough that
there was nothing left to find — not because this one stopped early.

## The roadmap was worse than stale in one place — it was wrong in both directions

Two research agents read `docs/roadmap.md` (1,100 lines) and the public `roadmap.json` (1,320 lines)
in full and cross-checked every unmarked item against the actual codebase. The public roadmap came
back clean — every row already had a note, and the four `partial`/`planned` rows checked out as
genuinely still not done. The internal one didn't: over 30 items marked `[ ]` were fully built —
Service Manager, Host Access Control, Password Strength Configuration, server-wide SSL/TLS policy,
phpMyAdmin, webhooks, Deploy from Git, Dynamic DNS, Site Publisher, Web Disk, Remote Database
Access, a guided database wizard, Visitors and Errors metrics, account-level API tokens, Team
Access itself — shipped in earlier sessions, never checked off. A few notes were wrong in the
opposite direction too: "API Tokens (Partial — admin-side only)" when account-level tokens had
existed for a session already; "no bulk SSL/TLS view" when a bulk view had shipped weeks earlier.
One item — Service Manager — genuinely tripped up an automated cross-check pass entirely: it exists
as `ServiceStatusController::control()` + `ControlServiceJob`, findable by grep for `service-control`
in the provisioning script, but invisible to a grep for "ServiceManager" by name. Worth remembering:
a stale-roadmap sweep has to search by what a feature *does*, not what its own roadmap line calls it.

Every correction cites the specific controller or service that proves it, not just a flipped
checkbox — the same discipline this file already asks of anyone marking something done for the
first time.

## Audit #08: the App Installer catalog, and `set -e` swallowing its own cleanup twice

The code-level audit was narrow in scope by comparison — eleven commits, all one feature, the App
Installer's rebuild from WordPress-only into a five-app catalog. Three findings, all fixed the same
day, two of them the same root cause wearing two different hats.

The first is the familiar shape: Nextcloud's admin email reaches `occ user:setting` as a bare
positional argument, not a `--flag=value` token, and unlike its two siblings used the same way
(`site_title`, `admin_user`), its validation didn't reject a leading dash — Laravel's own `email`
rule allows one in the local part. Fixed at both ends: the validation rule now matches its
siblings, and the `occ` call itself picked up a `--` end-of-options marker, since `occ` is Symfony
Console underneath and honors it. Belt and suspenders, the same standard this class of bug has been
held to since it first turned up in audit #06.

The second and third are the more interesting pair, because they were only found by actually
looking at the live server, not by reading the diff. Both come from the exact same mechanism: this
script runs under `set -euo pipefail`, and when a command inside an action fails, the script exits
*immediately* — any cleanup line written a few lines further down in that same action never gets a
chance to run. phpBB's install action writes a standalone YAML config file containing a real
database password (the only one of the five drivers that needs a separate credentials file at all,
since phpBB's installer takes a config file, not flags); when the installer itself failed during
this feature's own development — a real bug, since fixed — the config never got cleaned up, and sat
on `panel-dev` with a real password in it until this audit's own live-server check found it.
Nextcloud's archive download hit the identical shape from the opposite direction: this session's own
symlink-validation bug (also since fixed) rejected the downloaded archive mid-development, and the
280MB tarball was still sitting in `/tmp` — three of them, in fact — because `validate_tar_archive_safe()`
calls `fail()` internally, which exits before the caller's own `rm -f` two lines later ever runs.

Fixed differently on purpose. The phpBB config's cleanup is single-caller, so it got a local fix —
capture the installer's exit status through an `if` (exempt from `set -e`, unlike a bare command),
run the cleanup unconditionally, then fail afterward if it needs to. The archive cleanup is shared
by three drivers plus the account-import nested-archive check from audit #07, so it got fixed once,
at the source: `validate_tar_archive_safe()` now removes its own archive argument before every
`fail()` call it makes internally — correct for every current and future caller, not a patch
repeated three times. Re-verified against the same four hand-built test archives used to prove the
original audit #07 fix (two malicious symlinks, one traversal path, one safe archive shaped like
Nextcloud's real one) — every rejection now cleans up, the safe case still passes untouched.

Both leaked files were removed from `panel-dev` as part of the same live-server check that found
them.

## What didn't get fixed, on purpose

One finding from the live-server pass isn't a code fix at all: a disposable test server from
2026-08-02, self-documented in its own stored credentials as a one-shot install test that would "be
wiped and reinstalled again for future test runs" — still running five days later, on an old panel
build roughly forty versions behind current, fully exposed on the open internet across fifteen-plus
ports, with its own fail2ban confirming it's been genuinely probed (697 failed SSH attempts, 86 IPs
banned). No real accounts or customer data on it, and its password is a strong generated one, not a
default — so the practical exposure is bounded — but an old, forgotten, internet-facing instance of
the product is exactly the kind of thing that shouldn't just keep sitting there unnoticed. Flagged
in the report for a real decision (decommission, re-lock-down, or confirm it's still needed) rather
than acted on unilaterally — shutting down infrastructure isn't a call to make without asking.
