# Phase 149 - Asking Instead of Guessing

Our repair steps suppress error output by convention, which keeps them quiet but meant we had been inferring failures from side effects. For this investigation we added a diagnostic that runs the relevant operations with errors shown verbatim, plus a check for mandatory access control denials, so the next result would be an actual message rather than a deduction.

The lesson: when a check suppresses its own errors, add a way to see them while diagnosing.

Tested (2506/2506).
