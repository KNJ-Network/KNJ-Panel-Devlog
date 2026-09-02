# Phase 176 - The Subdomain That Never Got Its Address

During restore testing, an account with four subdomains had working DNS for only the first one. The records existed in the database but were missing from the generated zone file.

## Cause

When several subdomains are created in a loop, the zone file was built from data loaded before the later records were added. Re-reading the records from the database at build time fixes it.

## Verifying

On the development server, a restore with several subdomains now resolves every one. A regression test reproduces the stale-data situation directly.

Tested.
