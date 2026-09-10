# Phase 218 - The Column That Could Only Mean One Thing

The last two entries found the same bug wearing different clothes — a page that read
the account instead of the domain, so a second domain got nothing, or got the first one's
answer instead of its own. That version of the bug was a fixable habit: swap which object a
line of code asks. This time the habit had already been poured into concrete.

## When the fix isn't in the code

Six small tables each held one kind of setting — a custom error page, a MIME type, a
password-protected folder, a compression toggle. Every one of them had a column named
for the account. Not a shortcut taken in a hurry — a column is a decision made once and then
inherited by everything built on top of it: the unique constraint that stops a duplicate, the
relationship that loads the list, the query that writes the config file. Rewriting the code
that reads a column is an afternoon. Rewriting what the column *means* touches all four of
those at once, and none of them can be half-migrated without the other three going quietly
wrong.

## What a column promises

A unique constraint on the account column was making a real promise: you can only have one
404 page. For an account with one domain that promise is exactly true. For an account with
two, it was still being kept — just not the way anyone reading the settings page would guess.
The database enforced "one," counted correctly, and enforced the wrong "one." That's a worse
failure mode than an outright rejection, because nothing about it looks broken. The row exists,
the page renders, the count is one — it's just quietly describing the wrong domain's vhost the
entire time.

## Moving a promise without breaking it

Six tables needed the same operation: add a column for the real owner, copy every existing
row onto whichever domain it actually described when it was written, swap the constraint over,
and only then remove the column that had been making the wrong promise. The order matters more
than any single step in it — drop the old promise before the new one is fully in place, even
for an instant, and there's a window where nothing is actually being enforced at all.

The copying step carries its own kind of risk that's easy to under-test. It is trivially easy
to write a migration that works on an empty table and never actually prove what it does to a
real one — every row already there, not a fixture invented for the occasion, run through the
exact logic that will touch it live. That's the part worth building a real check for, not the
part that's satisfying to skip.

## Six tables, one answer, told six ways

Every table needed the same underlying repair, but never the same-shaped code. One setting
already listed several rows per account and only needed its ownership column swapped. Two
others had never been more than one row per account at all — enforcing "only one" was the
entire design, and widening what "one" means touches the public shape of the code that reads
it, not just its internal bookkeeping. A shared file picker, built once and reused by two
different settings pages, had been quietly anchored to the same wrong assumption the whole
time — fixed once, centrally, rather than patched twice in slightly different ways. And one
page had never organized itself by domain in the first place, so the honest fix wasn't a
picker at all — it was labeling each row with the domain it already belonged to and letting
them all show up together.

## What actually finished

Six settings that only ever worked for one domain now work for every domain an account has,
without six different explanations for why. The column that could only mean one thing now
means the right one — and the thing that used to quietly make one domain's setting look like
it belonged to another can no longer happen at all, not because the code is more careful, but
because the database itself won't allow it.
