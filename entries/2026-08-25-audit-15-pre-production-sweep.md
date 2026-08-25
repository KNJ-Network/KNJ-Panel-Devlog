# Phase 130 - Audit #15: A Toggle That Didn't Do What It Said

First audit run explicitly as a pre-production readiness check rather than only a post-feature diff —
Main, Mail Only, and two DNS Only servers are about to be stood up for real, replacing the current
ad-hoc fleet, so scope widened past the usual code review to a full compromise check across every
server currently running, not only the dev box. Five parallel, dimension-focused passes covering the 46
commits since Audit #14: WAF, CalDAV/CardDAV, Subscribers & Marketing Email, off-site backups, and the
remaining mail-hardening features (DNS import/export, mail autoconfig, per-mailbox quota/suspension,
disk-usage cleanup, virus scanning, email archiving).

## The real bug

`ClamavService::update()` sent the WHM form's `on_infected` value straight through to the provisioning
script exactly as validated and stored: lowercase `reject`/`discard`. The actual file it writes into —
`clamav-milter.conf` — needs the real ClamAV config syntax for that directive, which is capitalized:
`Reject`/`Discard`. The provisioning script's own case-sensitive check therefore rejected every real
save from the UI outright, so clamav-milter was never actually written or joined to Postfix's milter
chain — while the `Setting` row (and thus the checkbox on next page load) was persisted *before* the
script even ran, so it kept reading "enabled" regardless of whether the save had actually worked. Anyone
who'd turned this on since it shipped had every reason to believe mail was being scanned when it never
was.

Fixed on both ends: the value is capitalized only at the point it's written into the real ClamAV format
(keeping the stored/validated value lowercase everywhere else this codebase already expects it), and the
`Setting` now only updates once the script call has actually succeeded. Two new tests pin both halves —
the corrected value hits the wire, and a failed script call leaves the setting untouched.

## What else turned up

- **CalDAV/CardDAV had no brute-force logging on failed Basic Auth.** Every real calendar/contacts app
  sends Basic Auth on every single request — unlike a login form, there's no natural typing delay
  between attempts at all, and this backend never wrote a failure anywhere. The same
  `knjpanel-auth` fail2ban jail that already covers webmail and panel-login brute forces (the former
  fixed in Audit #14) never saw DAV traffic. Fixed by logging to the same channel/format those two
  already use — no new plumbing needed. Also added a regression test driving the real, fully-wired DAV
  server directly (bypassing `DavController::handle()`'s own `exit()` call, which would otherwise kill
  PHPUnit) to prove — live, against the actual DAVACL plugin, not from reading vendor source — that a
  mailbox genuinely can't reach another mailbox's calendar. One correction along the way: SabreDAV's own
  denial shape for a blocked `PROPFIND` is a `207 Multi-Status` with every property marked `403` inside
  it, not a bare top-level `403`/`404` — confirmed directly against the real plugin before the test's
  assertions were written, since the first draft assumed the wrong shape and the test caught it.
- **CSV formula injection in the new Subscribers export.** The public subscribe form takes a `name`
  with no charset restriction; a value like `=HYPERLINK("http://evil","click")` reached the account
  owner's exported CSV verbatim, and Excel/Sheets execute any cell starting with `=`, `+`, `-`, or `@` as
  a live formula on open. No prior feature in this codebase exports CSV at all, so this is a genuinely
  new pattern, not a repeat gap. Fixed by prefixing such values with a leading quote before `fputcsv`.
- **`SubscriberCampaignController` didn't re-verify the route's list id.** `show()`/`send()`/
  `sendTest()`/`destroy()` only checked the campaign's own ownership, never the `{list}` segment in the
  URL — every other action in the same controller does. Pairing your own campaign id with a victim
  account's list id leaked that list's *name* into the page title/breadcrumb. No subscriber data or
  write access exposed, but a real gap against this controller's own established convention. Fixed with
  the same `ensureOwnsList()` check the other three actions already had.
- **Three smaller Low findings, same visit:** `BackupDestination`'s encrypted `config` field was the one
  secret-bearing model in the codebase missing `#[Hidden]` (not reachable today, closed for consistency
  the same way Audit #13 did elsewhere); DNS zone import's apply step had no size cap where its own
  preview step already does; the public subscribe endpoint had no per-recipient throttle layered onto
  its existing per-IP one, so the same target email could be mail-bombed from many IPs or across hours —
  fixed with a new `subscribe-recipient` limiter keyed by `(list token, email)`.

## What's carried forward, not fixed

- **The WAF's CRS paranoia-level setting is stored and validated but never actually applied** —
  `crs-setup.conf` is copied verbatim from CRS's own example file at both install and ruleset-update
  time, so raising the setting does nothing (it also never lowers protection). The same "believed
  configured but isn't" shape as the ClamAV bug above, but applying it correctly means a real change to
  the from-source-installed engine already running on `panel-dev` — that needs its own build-and-verify
  pass, not a same-day fix bolted onto this one.
- **An `eval` in the WAF's nginx module build**, fed only by the box's own already-installed `nginx -V`
  output — no caller-supplied input reaches it, since the action takes no arguments. The eventual right
  fix is array-based argument handling, but nginx's captured configure-arguments string can itself
  contain embedded shell quoting that `eval` correctly re-interprets and naive word-splitting wouldn't —
  a mechanical swap risks quietly breaking the WAF build with no test harness to catch it. Deferred as
  its own piece of work.
- Everything else new this pass inherits the already-tracked write-then-validate-live pattern flagged as
  Low in Audit #14 — not a new gap, the same recurring design question.

## Clean

The full-fleet compromise check (all 7 servers, not just `panel-dev`) found no evidence of unauthorized
access anywhere — every login, open port, cron entry, and process traced to something that belongs
there. WAF's argv discipline, admin-only gating, and fail-closed behavior all held. Off-site backups:
genuinely encrypted at rest, no shell-out, no re-display route to leak a secret through even in
principle. DNS zone import/export can't inject anything manual entry would reject. Subscribers'
cross-account sweep came back otherwise clean — ownership enforced everywhere else, suppression checked
uniformly regardless of how a campaign sources its recipients. Git secret scan across all four repos:
nothing real, every apparent hit traced and confirmed fake or already-accepted.

## Verified

Full local suite (1,914 tests, up from 1,907 going in) and `pint --test` green after every fix. Deployed
to `panel-dev` via its own git-autodeploy; the corrected ClamAV value and the new DAV auth logging were
both confirmed live — a real wrong-password DAV request against a real disposable mailbox wrote a
`Failed login attempt from ::1 for info@test.knj.network` line straight into the log the existing jail
already watches. `panel-dev` is confirmed clean and ready to serve as the source for the first
production release cut.
