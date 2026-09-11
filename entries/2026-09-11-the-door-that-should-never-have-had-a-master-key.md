# Phase 230 - The Door That Should Never Have Had a Master Key

Every feature built into this panel so far has been about giving an operator more capability —
more visibility, more one-click fixes, more ways to help a customer without needing shell access.
Tonight's was the opposite kind of work: finding a capability that never should have existed in the
first place, and taking it back out.

## Convenient for staff, unbearable for everyone else

A hosting control panel's admin side has always had one honest job: run the server, help the
customers on it. Nothing about that job requires reading the contents of somebody's database.
Database Browser did exactly that anyway — every database on the box, listed, clickable, openable,
queryable with arbitrary SQL, no ownership check anywhere in the path. It wasn't built maliciously.
It was built because "let the admin see everything" is the easy default, the same shape as a master
key that opens every door in a building because cutting one key is simpler than fitting a hundred
locks. Convenient for whoever holds it. Catastrophic the moment anyone asks who else the building
belongs to.

The moment that mattered here wasn't abstract. It was a real screenshot: an account list that
happened to include a database belonging to someone the operator has an actual relationship with,
and the realization that clicking it open was one click away, with nothing in the system's design
stopping it.

## Two different shapes of the same mistake

Access Hosts turned out to be the same problem wearing a subtler disguise. It didn't read data
directly — it granted the *ability* to read data, letting an admin open a remote connection to any
customer's database on their behalf, without asking them. Not a locked door held open. A spare key
cut for someone else's lock, kept in a drawer an admin could reach into any time. Different
mechanism, same failure: a decision that belongs to the data's owner, made available to someone who
isn't.

Neither of these was an access-control bug in the usual sense — nothing was misconfigured, no
permission check was missing where one was expected. The whole feature was the mistake. There's no
patch for a door that shouldn't be a door.

## Taking out a wall without dropping the ceiling

The one complication: a legitimate feature had been built standing on top of the one being removed.
Repairing and optimizing a database is real maintenance work, the database equivalent of defragging
a disk — it touches structure, never content, and any operator genuinely needs it sometimes. But its
only trigger lived on Database Browser's own per-database page. Deleting that page wholesale would
have quietly deleted a legitimate tool along with the illegitimate one it happened to be standing
next to.

The fix wasn't to keep a smaller Database Browser around to preserve that one button. It was to ask
what that button actually needed — a name, nothing else — and give it exactly that, moved onto its
own page, with no path left anywhere back to a table's contents.

## What actually finished

An admin can no longer open a customer's database, read a single row, or hand themselves remote
access to one, from anywhere in this panel. A database repair still works, using only the one fact
it ever needed — that the database exists — never the one fact it never should have had.
