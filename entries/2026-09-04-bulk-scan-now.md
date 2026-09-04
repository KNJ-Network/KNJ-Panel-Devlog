# Phase 201 - One Click Instead of One Account at a Time

A small gap, found the same way most of the real ones get found on this project: a screenshot of
the live server, sent in without warning. The Malware Scan Overview page — stats, security status,
recent findings — had no way to actually start a scan from it. The only trigger anywhere in the
panel was a single "Scan for Malware" row action, one account at a time, on the full accounts list.
Checking a handful of accounts after installing something new meant clicking through each one in
turn.

## What "select some accounts and go" actually needs

The scan itself was never the hard part — dispatching a scan job is a cheap queue push, and the
existing single-account action already did exactly that. The real work was a UI pattern this panel
had never needed before: a checkbox list living inside a popup, not a full page of its own. Every
existing modal in the panel is the shared confirm-or-cancel dialog — fine for "are you sure," no
room for a real form. The new one is scoped to its own page instead, built the same way the rest of
this codebase handles anything else that doesn't need a dependency: two small event listeners
keeping a "select all" checkbox and the individual rows in sync in both directions, checking one off
un-checks "select all," checking the last one back on re-checks it.

Submitting posts a plain list of account ids to a new endpoint that runs the same
`MalwareScanRun` + dispatched job loop the single-account action already used, just once per
selected account instead of once total. Nothing about the actual scanning changed — only the number
of times it gets triggered by one click.

## Two ways a selection can go stale between opening the modal and submitting it

The eligible-accounts list — who's even allowed to show up as a checkbox in the first place —
already existed as a concept, just inline in the single-account flow. Anything without a real,
stable home directory to point a scan at (not provisioned yet, provisioning failed, mid-deletion) is
excluded. A plain suspension or a security suspension is deliberately still included — re-scanning a
security-suspended account is exactly how it gets reactivated, so leaving it off the list would have
broken the one feature that most needs it.

That eligibility check runs twice, not once — once to decide what shows up in the modal, and again
inside the submit handler itself. A modal can sit open in a tab while the world underneath it keeps
moving: an account could start deleting, or already be mid-scan from something else entirely, in the
gap between page load and form submit. Re-deriving both exclusions fresh at submit time, rather than
trusting whatever the browser happened to send back, means a stale selection just quietly skips the
accounts that no longer qualify instead of either erroring out or scanning something it shouldn't.
The status message says so plainly — "Started 2 scans. 1 account already had a scan in progress and
was skipped" — rather than pretending everything requested actually ran.

Tested (2859/2859), pint clean, verified live on panel-dev: opened the modal, exercised the
select-all sync both directions, submitted a real bulk scan against two disposable test accounts and
watched both `MalwareScanRun` rows land as genuinely `running`, then confirmed the skip path by
resubmitting one of those same accounts while its scan was still in flight.
