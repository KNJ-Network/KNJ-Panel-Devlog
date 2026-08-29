# Phase 146 - One of Two, and Only One of Two

The diagnostics showed that the preconditions for our repair were all satisfied, and that one of the two directories it touches had been updated while the other had not. That pointed away from "the repair did not run" and toward something specific about one directory.

We considered a few possible explanations, such as special filesystem attributes or mount boundaries, and extended the read-only diagnostics to check for them. No behavior changed in this step.

The lesson: look at the actual state before writing a fix for a state that was only assumed.

Tested (2506/2506).
