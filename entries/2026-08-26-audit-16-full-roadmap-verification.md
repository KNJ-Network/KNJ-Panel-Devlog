# Phase 134 - Audit #16: Verifying Every "Live" Row Is Actually True

A different kind of audit from the fifteen before it. Every earlier one reviewed new code written since
the previous pass. This one went back and checked the public roadmap itself: all 178 rows marked
"Live", across the admin and customer sides, each against the real code rather than against memory.
Reviewers took slices and had to point at the actual code for every claim.

It took twenty-two rounds, because each round taught us how to look for the next. The short version
follows.

## What it found

The first round found one feature whose per-login limits were not being applied as intended. It was
fixed with its own tests before anything else. Alongside it were three other problems and seven
roadmap lines that had drifted out of date. The roadmap now says what the code does.

Then a pattern showed itself. Across many unrelated features, the panel would record that something had
succeeded before confirming that the server had actually done it, and if the server step failed, the
panel kept saying all was well. Nobody wrote that on purpose; it had grown as a habit. The technique
that found the long tail was simple: for every "create" or "turn on" action, check its "delete" or
"turn off" twin, which had usually been overlooked. Applied systematically, it closed the pattern out.

Two findings stood out. In one case, a rule was marked as removed in the panel before the server
confirmed it, so a failed removal left the two disagreeing. The other was a gap in a deployment
template that the test suite could not see, because it lived outside code.

## A different kind of bug, and a new way of looking

After the fourteenth round came back clean, one more pass found two bugs of a kind the earlier
technique could not have caught: places where an unexpected error skipped the code meant to handle
failure. The lesson is that a codebase can look clean by one method of checking and still hide
problems from another.

So the later rounds swept by kind of problem across the whole codebase rather than by feature:
unhandled errors, jobs that never finish, simultaneous clicks, injection, and access control between
accounts. The access-control and injection sweeps, the two that matter most for a hosting panel, came
back clean.

## Asking whether it was really done

Rounds got narrower, down to three findings in the last, and a deliberate design pass closed the one
gap that needed real thought: removing an account had not checked whether server-side clean-up
actually succeeded. A failure is now visible and retried.

The audit ended with 2,446 passing tests, a clean run of the code-style checks, and the result
deployed and checked on our development server.
