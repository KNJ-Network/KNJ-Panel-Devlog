# Phase 70 - A Roadmap That Was Stale in Both Directions, and Two Cleanup Bugs

Same approach as audit #07: stop, check everything, and don't move on until it's clean. This
time the scope went wider: a full roadmap accuracy sweep first (every unmarked item cross-checked
against the real code, every public roadmap row checked for a missing note), then the audit
itself, then a cross-check across every repository.

## The roadmap was wrong in both directions

Two review passes read the internal roadmap and the public roadmap data in full and cross-checked
unmarked items against the codebase. The public roadmap came back clean. The internal one didn't:
over 30 items marked as open were fully built, shipped in earlier work and never checked off. A
few notes were wrong the other way too. One item, Service Manager, tripped up an automated check
because it exists as a status-page action plus a queued job, findable by what it does but not by
name. A roadmap sweep has to search by what a feature does, not what its roadmap line calls it.

## Audit #08: the App Installer catalog

The code-level audit was narrow: eleven commits, all one feature, the App Installer's rebuild into
a multi-app catalog. Three findings, all fixed the same day.

The first was an input-validation gap for an installer's admin email, which didn't match the
validation its sibling inputs already had. It was fixed at both ends.

The other two share one root cause, found only by looking at the live development server: the
provisioning script stops immediately when a command fails, so cleanup lines further down in the
same action never ran. During this feature's own development, that left a temporary installer
config and several downloaded archives behind on the development server. Both were removed, and
the fix was made at the source: shared archive validation now cleans up after itself before every
failure, and the one-off config cleanup runs unconditionally. The fix was re-verified against the
same set of hand-built test archives used to prove the audit #07 fix.

The cross-check also noted an old disposable test server from earlier in the month that needed a
decision about whether to keep it. That was handled as an infrastructure housekeeping item.
