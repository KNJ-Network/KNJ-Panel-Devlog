# Phase 216 - The Tools That Forgot There Were Others

Three tools on the same page, each with its own quiet assumption: that an account only ever has
the one domain. True the day each was built. Not true for a while now.

## Bought on purpose, useless on arrival

The domains existed for a real reason — bought deliberately, to redirect back to the main one, the
plain and ordinary reason anyone buys a second address for something they already have a first
address for. Added them the way the page says to add them. Watched them land in their own folder,
exactly as promised.

And then nowhere to tell them what to do next. The page that redirects a domain only knew about one
domain, and it wasn't either of the new ones. The page that edits a domain's own DNS only knew about
one domain, and it wasn't either of the new ones. Both pages had been right, once, back when "the
domain" and "the account's only domain" were the same sentence. Neither had noticed when that
stopped being true.

## The tool that answered a question nobody was asking any more

Ask a single-domain tool to work for a second domain and it doesn't error, doesn't warn, doesn't
even seem to notice anything's wrong. It just quietly keeps answering about the first one. Every
field, every save button, every "successfully updated" message — all real, all functioning exactly
as designed, all aimed at an address that was never the one just typed into the form above it. The
UI never lies here. It also never once mentions the question it was actually asked.

That's a harder bug to trip over than an error message, because nothing about using it feels wrong.
The redirect page redirects. It's just always the *first* domain that moves, no matter which one the
new domains, sitting one page over, were bought to move.

## Fixed by remembering there's more than one

Both tools now take the domain as a real question, not an assumption — a selector where more than
one answer exists, a link that already knows which one it's pointing at instead of only ever knowing
the one it was born with. The redirect control moved onto the page that already lists every domain
by name, so there's no longer a separate page quietly speaking for whichever one it was written for
first.

## The one thing this account never had at all

Somewhere in the same conversation, a smaller, plainer gap: nothing anywhere let a domain's own
folder be changed once it existed. Set once, at the moment of creation, and never touched again by
any button on any page — not broken, just never built. If two domains were ever going to share the
same content without one of them redirecting to the other, there was no way to say so.

That's in now too, offered everywhere except the account's very first domain, which stays fixed on
purpose — too much else about the account already assumes it never moves. Everywhere else, it's a
plain, direct answer to a plain, direct question: point this folder at that one. Not a workaround.
Just the feature that was never actually there to ask for in the first place.
