# Phase 180 - The Page That Was Never Going to Load

The admin dashboard on a satellite server showed a spinner indefinitely. The page shell loaded instantly, but a request for live service status never returned while every other request finished quickly.

## Cause

The status check queried services one after another with no explicit timeout, so a single stuck query could take far too long. A failure also discarded every other result along with it.

## The fix

Each check now has its own short timeout and its own recovery. A stuck service shows as inactive for that row only, and every other row shows its real status. A partial failure now looks like a partial failure.

## Verifying

A regression test makes one check hang and asserts every other result is unaffected. On a live server of the same class, all applicable services report correctly.

Tested (2709/2709).
