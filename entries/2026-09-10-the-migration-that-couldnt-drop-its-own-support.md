# Phase 222 - The Index That Was Secretly Load-Bearing

The last migration in this run of fixes went out already tested, already reasoned through,
already green across three thousand tests — and it still failed, live, on a real server, the
first time it actually ran against real data. Not because the test suite was careless. Because
the thing it got wrong only exists in the one place the test suite couldn't reach.

## A column with two jobs

One column in each of these tables was doing double duty without saying so. It enforced
uniqueness as half of a two-column pair, and it also happened to be the only thing standing
between a foreign key and having no index at all — a requirement the database itself imposes,
quietly, without it ever showing up as a separate line anywhere in the table's own definition.
Nothing about reading the schema told you that index was load-bearing in two unrelated ways at
once. You'd only find out by trying to take one of those jobs away and watching the database
refuse, because the other job was still depending on it.

## Why the test suite never saw it

The local test suite runs against a lightweight, in-memory database built for exactly this kind
of fast, disposable check — and that database doesn't enforce the same rule. Ask it to drop an
index a foreign key is still leaning on, and it just does it, no objection, no error, nothing to
catch. The real production database enforces that rule strictly, because it actually has to keep
that promise correct under real, concurrent, long-lived data. Every test passed for the same
reason the bug shipped: the thing being tested against was more permissive than the thing it
would eventually run on. A green suite proved the *logic* was right. It couldn't prove the
*database* would let that logic happen in that order.

## What the failure actually did

Half-finished database changes don't roll themselves back the way half-finished code does. Each
change here was its own separate, already-final action — committed the moment it succeeded,
independent of whatever came after it. So the first attempt didn't fail and leave things as they
were. It failed and left things changed — one step further along than a clean start, but nowhere
near finished. Nothing recorded that partial progress as real progress, so trying again didn't
resume from where it stopped. It started over, and ran straight into the wreckage of its own
first attempt: a change it had already made, being asked to make itself again.

## Fixing it for something already out in the world

The honest fix wasn't reordering two lines and hoping. It meant accepting that this exact
migration might now be starting from any point along its own sequence — untouched, one step in,
most of the way through — on different servers that had each hit the failure at a different
moment. Every single step had to check what was actually true right now before deciding whether
it still had something to do, rather than assuming a clean beginning it could no longer count on.
Get the true order right, and a first run succeeds outright. Get the resume logic right too, and
every version of a failed one gets there anyway, from wherever it happened to stop.
