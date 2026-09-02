# Phase 175 - The Password That Broke Its Own Config

A restore test produced a WordPress site with a PHP parse error. The database and the password were fine; the configuration rewrite was not.

## Cause

The rewrite used a regular-expression replacement, and replacement strings treat a dollar sign followed by a digit as a reference rather than literal text. A generated password containing that sequence was altered when written.

## The fix

We switched to a callback-based replacement, which writes the string exactly as given. The regression test uses a password with that sequence, since ordinary passwords pass both before and after the fix, and a syntax check of the resulting file confirms validity.

Tested, and verified against the archive that surfaced it.
