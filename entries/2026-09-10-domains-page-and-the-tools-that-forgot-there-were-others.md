# Phase 216 - The Tools That Forgot There Were Others

Three tools on the same page each carried a quiet assumption that an account only ever has one domain.
That was true when each was built, and not for a while now.

## Added on purpose, useless on arrival

Consider an account with additional domains that exist only to redirect to the main one. They are added
the way the page says to, and land in their own folder as promised. Then there was nowhere to tell them
what to do next. The page that redirects a domain only knew about one domain, and the page that edits a
domain's DNS likewise. Both had been right when "the domain" and "the account's only domain" were the
same thing.

## The tool that answered a question nobody was asking any more

Ask a single-domain tool to work for a second domain and it doesn't error or warn. It keeps answering
about the first one. Every field, save button and "successfully updated" message is real and works as
designed, aimed at an address that wasn't the one just chosen. That is a harder bug to trip over than an
error message, because nothing about using it feels wrong.

## Fixed by remembering there's more than one

Both tools now take the domain as a real question: a selector where more than one answer exists, and a
link that knows which domain it points at. The redirect control moved onto the page that already lists
every domain by name.

## A gap in folder control

Nothing let a domain's folder be changed after creation. If two domains were to share content without one
redirecting to the other, there was no way to say so. That is now possible for every domain except the
account's first, which stays fixed on purpose because too much else assumes it never moves.
