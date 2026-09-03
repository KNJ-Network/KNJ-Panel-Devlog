# Phase 181 - The Filesystem Nobody Was Watching

On a test restore, an account's dashboard showed 1MB used although its files totalled about 1.8GB. On this server the home area was on a separate disk, and the quota logic had assumed it shared one filesystem with the OS.

## Fix

The quota tooling now detects a separate home filesystem and enables and reads quotas for it. New installs handle this at setup. Existing servers use an idempotent self-heal action wired into the existing scheduled usage refresh, so no admin action is needed.

Usage reports now cover two filesystems, so the parser sums per-user entries instead of letting the last one win. A regression test covers it, and the self-heal correctly does nothing on a server without a separate home filesystem.

Tested (2709/2709).
