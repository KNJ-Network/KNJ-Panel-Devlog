# Phase 181 - The Filesystem Nobody Was Watching

A restored account's dashboard said 1MB used. The account's own real files — a real WordPress site,
images and all — added up to 1.8GB, sitting right there on disk, visible to anyone who looked. Not a
rounding error, not a caching lag. The number was reading from a place that had simply never been
told those files existed.

## Two disks, one question

Every hosting account's files live under `/home/<username>/...`. On most of this panel's servers,
`/home` is just a directory — part of the same filesystem as everything else. On this one, it wasn't.
A quick check of the actual mounts showed two genuinely separate block devices: one for `/`, a
completely different one, a completely different size, for `/home`. That split is legitimate and
common — bigger storage for hosting data, kept apart from the OS disk — and this panel already had
code elsewhere that specifically detects it, for a completely different reason (splitting the
dashboard's own disk-stats display). The quota system had never heard of it.

## Enforcement, not just a number on a page

Real disk quotas on Linux work per filesystem. Enabling them on `/` says nothing about `/home` — the
kernel has to be told, separately, for each one. The provisioning script's own quota-reading and
quota-setting commands both hardcoded `/` unconditionally, going all the way back to the very first
version of the quota feature. On a server where `/home` is its own filesystem, that's not just a
reporting gap — it's an enforcement gap. The kernel was never told to track usage there at all, which
means it was never told to *block* a write past the limit either. A package's disk cap, the thing
customers are actually paying for and the thing this panel already advertises as "enforced by the
server itself," simply wasn't being enforced for any account whose files happened to live on the
separated disk.

## Fixing it without anyone touching a terminal

New installs are easy — teach the bootstrap script the same separate-mount detection this codebase
already uses elsewhere, and enable quotas on `/home` too, right alongside the existing `/` setup.
Already-running servers are the harder case: a script that only runs once, at first setup, does
nothing for a server that's already live. The fix there isn't a manual step for an admin to remember
— it's a new, idempotent self-heal action, wired into the same scheduled job that already refreshes
every account's disk usage every fifteen minutes. The next time that job runs anywhere it's needed, it
quietly turns quotas on, checks the existing files, and starts tracking correctly — no SSH, no admin
action, nothing to forget. Exactly the same philosophy this panel already holds itself to everywhere
else: the product manages its own server state, the operator shouldn't have to.

## Verifying it against a real gap, not an assumption

The usage-report parsing had its own smaller trap once both filesystems are being read: a report now
carries two separate blocks, and the existing code treated a repeated username as "last one wins"
rather than "add them together" — silently losing whichever half came first. Fixed to sum, with a
regression test built from two real repquota-shaped blocks for the same user. Live-verified on the
dev server too, in the other direction: confirmed the self-heal action correctly recognizes a server
where `/home` genuinely isn't separate, and does nothing at all rather than acting on a filesystem
split that was never actually there.

Tested (2709/2709).
