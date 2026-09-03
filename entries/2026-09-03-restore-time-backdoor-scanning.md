# Phase 184 - Teaching the Restore Itself to Look

Our earlier review of a problematic backup was manual, which is not a repeatable safeguard. This phase makes the restore check for it automatically.

## A gap in signature scanning

The existing malware scanner is signature-based, so it recognizes known code and not custom code. A restore-time check complements it.

## Keeping the rule narrow

A broad rule would train admins to ignore it, so the check targets one specific pattern. Before trusting it, we tested it against a full real-world archive and its databases with no false alarms. A flagged file is simply not restored and the log says what was skipped and why. A flagged database row is reported for a person to decide, because automatically editing a live SQL dump risks corrupting nearby data.

## Where it runs

It runs just before the privileged copy step, so a skipped file never arrives on the account and needs no cleanup.

## Verifying

Tests confirm a reconstructed example is flagged and ordinary files are left alone, and the same held on the deployed code.

Tested (2724/2724).
