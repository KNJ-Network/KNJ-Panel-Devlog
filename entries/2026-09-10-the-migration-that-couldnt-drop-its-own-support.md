# Phase 222 - The Index That Was Secretly Load-Bearing

A database migration passed the full test suite, over three thousand tests, and still failed the first
time it ran against real data on a development server. The suite wasn't careless; the problem only exists
somewhere it can't reach.

## A column with two jobs

One column in each affected table was doing double duty without saying so. It took part in a uniqueness
rule and also happened to be what a relationship constraint relied on, a requirement the database
enforces without it appearing as a separate line in the table definition. You only find out by removing
one of those jobs and watching the database refuse.

## Why the test suite never saw it

The tests run against a lightweight in-memory database that doesn't enforce that rule, while the real
database does, strictly. Every test passed for the same reason the problem shipped: the thing being
tested against was more permissive than the thing it would run on. A green suite proved the logic was
right, not that the real database would allow it in that order.

## What the failure did

Each schema change commits as soon as it succeeds, so a failure part-way doesn't roll back. The first
attempt left things one step further along, and nothing recorded that, so retrying started over and hit
its own earlier changes.

## Fixing it for something already out in the world

The fix wasn't reordering two lines. The migration must be able to start from any point in its own
sequence, so each step now checks what is actually true before deciding whether it has work to do. With
the right order, a first run succeeds, and with the resume logic, a partially applied one completes too.
