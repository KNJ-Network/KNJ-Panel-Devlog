# Phase 201 - One Click Instead of One Account at a Time

A small gap, found while reviewing the live admin pages. The Malware Scan Overview page had stats,
security status and recent findings, but no way to start a scan from it. The only trigger was a
single-account "Scan for Malware" action on the full accounts list, so checking several accounts meant
clicking through each in turn.

## What "select some accounts and go" needs

The scan itself was never the hard part, and the single-account action already started one. The real
work was a UI pattern this panel hadn't needed before: a checkbox list inside a popup. It is built the
same way the rest of the codebase handles anything that doesn't need a dependency, with "select all"
and the individual rows kept in sync in both directions. Submitting starts one scan per selected
account, using the same path as the single-account action. Nothing about the scanning itself changed.

## Two ways a selection can go stale

Only accounts that can actually be scanned appear as choices. A plain suspension or a security
suspension is deliberately still included, because re-scanning a security-suspended account is how it
gets reactivated, and leaving it off would have broken the feature that most needs it.

That eligibility check runs twice: once to build the list, and again when the form is submitted. A
modal can sit open while the world underneath it changes, so a stale selection now skips accounts that
no longer qualify rather than erroring or scanning something it shouldn't. The status message says so
plainly, for example "Started 2 scans. 1 account already had a scan in progress and was skipped",
rather than pretending everything requested ran.

Tested (2859/2859), pint clean, and verified on the development server with disposable test accounts,
including the skip path.
