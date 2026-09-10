# Phase 218 - The Column That Could Only Mean One Thing

The last two entries found the same bug in different clothes: a page that read the account instead of the
domain, so a second domain got nothing or got the first one's answer. That version was a fixable habit.
This time the habit had been built into the data structure.

## When the fix isn't in the code

Six small tables each held one kind of per-site setting, and each was keyed to the account. A column is a
decision made once and inherited by everything built on it: the uniqueness rule, the relationship that
loads the list, and the code that writes configuration. Changing the code that reads a column is an
afternoon; changing what the column means touches all of those at once.

## What a column promises

A uniqueness rule on the account was making a promise: one setting of that kind. For an account with one
domain that is exactly true. For an account with two, the database enforced "one" correctly but the
wrong "one", which is worse than an outright rejection because nothing looks broken.

## Moving a promise without breaking it

Each table needed the same operation: add a column for the real owner, copy every existing row onto the
domain it described, switch the rule over, and only then remove the old column. The order matters more
than any single step, since dropping the old rule before the new one is in place leaves a window where
nothing is enforced. The copy step was checked against real existing rows rather than only an empty
table.

## Six tables, one answer, told six ways

The same repair needed different code in each place: some settings already allowed several rows per
account, some had been one row by design, a shared file picker had the same wrong assumption and was
fixed once centrally, and one page was relabeled by domain instead of getting a picker.

## What actually finished

Six settings that only worked for one domain now work for every domain an account has, and the database
itself prevents one domain's setting from being attributed to another.
