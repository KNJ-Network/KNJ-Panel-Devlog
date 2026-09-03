# Phase 185 - The Tool That Checked Itself

A new restore-time safeguard shipped the same day it was designed, and within hours we found it produced a false positive on a core WordPress file. The unit tests had only covered hand-picked cases; the decisive test was running it against a complete real-world archive.

## The lesson

A rule that flags a combination of behaviours must ask whether they occur together, not merely somewhere in the same large file. We tightened the rule accordingly, so ordinary large files are left alone.

## An unexpected benefit

Re-running against the same archive also found additional cases in our own test material that the initial manual review had missed, which is exactly the argument for automated checks over hand searching.

## Verifying

We checked both directions of failure: a large ordinary file is left alone, and a rebuilt version of the real pattern is still caught.

Tested (2726/2726).
