# Phase 187 - A Wrong Answer Beats a Random One

The last incident this week left a real lesson behind that hadn't been acted on yet: a domain whose
certificate never issued didn't just go without HTTPS. It went without HTTPS in a way that let
someone else's website show up under its name.

## What "no certificate" actually meant

The failure mode was never announced anywhere — nothing crashed, nothing logged an error a person
would notice. A browser following a site's own request to switch to HTTPS simply landed on whatever
the server happened to answer with by default when nothing else claimed that exact address. Most of
the time that's harmless, because most domains do have a certificate. The one time it wasn't harmless
was the one that got noticed: a fresh account whose address hadn't finished spreading to the wider
internet yet, hit at exactly the wrong moment, ended up momentarily showing a completely unrelated
site's content under its own name.

## Refusing to leave the question unanswered

The honest fix isn't "make certificate issuance never fail" — DNS propagation delay, a typo, a
domain pointed somewhere else entirely: those are going to keep happening, permanently, and no amount
of retrying changes that. The actual gap was smaller and more fixable than it looked: the server
always had *an* answer ready for every address it didn't recognize. It just never had one specifically
prepared for an address it *did* recognize but couldn't yet vouch for.

That's the whole shape of the fix — when a real certificate can't be issued, generate a temporary one
anyway, specifically for that one domain, and use it instead of falling through to whatever else
happens to be listening. It won't be trusted. A visitor gets a warning telling them exactly that. But
the warning is honest, and it's attached to the correct site — worlds apart from an unrelated site's
real content answering silently in its place.

## Automatic, not a chore added to someone's list

A temporary certificate that just sits there is still a problem, only a quieter one. The fix checks
back on its own, on a short cycle, and swaps in the real thing the moment it's actually available —
which, for the exact kind of failure this was built around, is usually just a matter of minutes. Not
something anyone has to remember to look for.

## Not wasting the parts that already worked

Retrying doesn't mean starting over. A domain that's still failing keeps its already-generated
temporary certificate rather than being handed a fresh one for no reason every time. And when a real
certificate finally does land, the temporary configuration it's replacing gets removed cleanly first —
never left sitting alongside the new one, which is exactly the kind of leftover that quietly breaks a
server's configuration the next time anyone touches it.

## Proving it against a domain that would actually fail

Verified against a real account on a real server, using a domain deliberately picked because nothing
would ever resolve it — the exact condition this whole fix exists for. Every real step ran for real:
certificate generation, the server configuration change, a reload, and then an actual browser-style
connection to confirm what came back. What came back was the right site, with its own name on the
warning, not someone else's content wearing it. Retrying against the same still-unreachable domain a
second time changed nothing that didn't need changing — same certificate, same configuration, still
valid.

Tested (2758/2758).
