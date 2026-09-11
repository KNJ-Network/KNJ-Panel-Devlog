# Phase 229 - What Moving Out Forgot to Take

Verifying the last fix meant actually using it — create a test subdomain, give it real records,
confirm they land correctly, then clean up afterward the way every test fixture on this project
gets cleaned up. The cleanup step is usually the boring part. Tonight it wasn't.

## Removal remembered the wrong inventory

Deleting a subdomain has always meant deleting one thing: the address record that pointed its name
at the server. That was a complete inventory for as long as an address record was the only thing a
subdomain could ever have owned. It stopped being complete the moment a subdomain could also own
SPF, DKIM, and DMARC records of its own — new belongings, added by a brand-new feature, that the
old moving-out checklist had never been told to look for.

Nothing about the removal code was wrong for what it knew about. It just didn't know about the new
things yet, because those things didn't exist when it was written. That's the ordinary shape of
this kind of bug: not a mistake, a checklist that quietly went out of date the moment a sibling
feature expanded what could be there.

## A second bug hiding inside the fix for the first

Writing the fix meant matching four different record shapes instead of one, and the natural way to
write that is a chain of "or this, or this, or this." That natural way is also a trap here: added
carelessly, an "or" doesn't nest inside the existing "and this is the right zone" boundary — it
sits next to it instead. The result would have been a delete that reached past its own subdomain
and into whichever other zone happened to have a record with the same name. An amateur mistake,
almost made by someone who should know better, caught only by stopping to ask what the actual SQL
underneath the convenient syntax was going to say.

The fix for that isn't cleverness — it's grouping. Put every "or" inside its own parentheses first,
so the zone boundary still wraps the whole group instead of just the first clause. The kind of
thing that's invisible in the code and completely different in what it actually does.

## What actually finished

Removing a subdomain now takes everything that subdomain came to own, not just the one thing it
happened to own when the removal code was first written — and takes only that subdomain's own
things, nothing borrowed from a neighbor by an unlucky trick of query syntax.
