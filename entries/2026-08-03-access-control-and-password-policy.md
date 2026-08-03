# Phase 48 - A Firewall Bug That Only Failed Through the Real App

Two items shipped in the Security section: per-service IP allow/deny lists ("Access Control") and a
central, admin-configurable password policy. Scoped API tokens and external login were still in
progress.

Password Policy was the straightforward one: a single central policy now builds the password
validation rule everywhere a password is set, replacing a hard-coded minimum length that had been
copied across roughly ten controllers. Defaults match what was already enforced, so no existing
install changes until an admin turns on stronger requirements.

Access Control — restricting Panel Login, SSH, FTP, Mail, or DNS to specific IPs at the firewall —
was a different story. It worked every time it was tested by hand, and failed intermittently when
triggered through the real running app, with no error anywhere. Chasing it meant ruling out one
plausible theory after another, each patched, each still leaving the bug in place.

The cause turned out to be a small detail in how the allow-list was written to disk and read back,
which hand-written test files never reproduced. It was fixed with the standard idiom for that case
and verified with several full cycles through the real UI in both directions.

The defensive extras added while chasing the wrong theories — serializing firewall writes and
verifying every rule actually applied before reporting success — stayed in as real insurance.
