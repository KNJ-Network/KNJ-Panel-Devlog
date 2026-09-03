# Phase 188 - The Shortcut That Called Itself

A customer reported his own domain unreachable on the port that was supposed to make it feel like
his. Not a timeout, not a certificate warning — a real page, from a real web server, saying the
request itself was too big to answer. Nothing about that request had grown. Something between it
and the panel had been quietly making it bigger, one hop at a time, without ever finishing.

## A shortcut that only sounds like a shortcut

The feature behind the broken address exists to make a hosted domain feel like it belongs to its
owner, not to the panel renting it space: visit your own domain on the classic port, and you land on
your own login screen, on your own certificate, without ever seeing the panel's shared hostname. To
pull that off honestly, the request still has to reach the same backend every other account shares —
so behind the scenes, it loops back through the server to itself.

That loop needed one more piece of information than it was given: which of many identical-looking
doors to knock on when it got there. It was told to knock using the name on the door it had just
walked through. For most domains that's harmless — nothing else on the other side answers to that
name. But this customer's domain had its own private entrance too, listening on the exact same
corridor the loop was walking down. Knock using your own name, and you don't reach the shared lobby.
You reach your own front door again. Which loops back through the same corridor. Which knocks on
your own front door again.

Every trip round added a little more to what the request was carrying, the way a game of telephone
played entirely by one person gets a little longer each time it repeats itself. Eventually there was
too much to carry through the door at all, and the building's own front desk turned it away outright
— to the visitor, indistinguishable from the request simply being rejected.

## The name that can't collide

The fix isn't to stop the loop from happening — the loop is how the feature works at all. It's to
give it a name that can never belong to more than one door: the panel's own name, not the visitor's.
That name is guaranteed unique because it's the one thing on the whole server that only ever answers
to itself. The visitor's actual domain still needs to travel with the request — cookies, and the
occasional link the panel generates on its own, both depend on knowing where the visitor really came
from — so it now rides along as a separate, clearly-labeled note instead of standing in for the door
name itself. Same information, no longer capable of being mistaken for an address.

The same repair applies in two more places than the one that got reported. The customer's own
memorable web address for reaching webmail — no port number to remember, just the domain itself —
turned out to walk through an identical corridor with an identical risk, quietly, for every domain on
the server, not only ones an admin had happened to test.

## The bug the loop had been hiding

Stopping the loop let a second, unrelated mistake become visible for the first time. The rule for
"which page did you actually ask for" had been reading the wrong half of its own answer — grabbing an
empty placeholder instead of the part that actually held it — so any request past the shortcut's own
front page had silently been flattened back down to that front page regardless of what was actually
typed. Nobody had noticed, because nothing had ever gotten far enough down the corridor to try asking
for anything else.

A third mistake surfaced the same way, once the first two stopped hiding it: the note carrying "which
port did this request really arrive on" was being written in a shape the framework simply doesn't
read — a lone number, where it expects a full address with the number attached. Silently ignored, it
meant a login attempt on the classic port would forget it was ever there, and hand the visitor back to
whatever ordinary address answers by default instead — which, for a hosting customer, is their own
website. Not broken exactly. Just no longer where they'd asked to go.

## Proving the corridor doesn't loop anymore

Reproducing this safely meant putting a disposable domain into exactly the state the real one was
found in — the same self-addressed door, the same shared corridor — then confirming the repair
finds and rewrites it on its own the next time anything routine touches that domain, no manual
intervention, no re-provisioning from scratch. It did. A request that used to come back as a rejected,
oversized mess now comes back as an ordinary login redirect, staying on the same domain, the same
port, the whole way through.

Tested (2774/2774).
