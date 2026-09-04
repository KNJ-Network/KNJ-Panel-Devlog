# Phase 194 - Standing In Someone Else's Shoes, Correctly

The routing fix from earlier the same day worked. Double-checking it, though, by stepping into a
real customer's account the way support staff sometimes need to — seeing exactly what they see,
without needing their password — turned up something else entirely. Clicking the same button
appeared to do nothing at all. No error. No QR code. Nothing.

## Two ways of being someone else

A system that lets one person temporarily see the world through someone else's account has to
answer a quiet but important question every single time: for the rest of this request, who is
"the current user"? Get that answer right everywhere, and stepping into someone else's shoes is
seamless — every page shows their real data, every action affects their real account, and the
person doing the standing-in never has to think about it. Get it wrong in even one place, and an
action silently lands on the wrong account instead of failing loudly, which is a far worse outcome
than an obvious error.

This system had already gotten that right almost everywhere, deliberately keeping the helper's own
real login completely separate underneath, so that stepping into someone else's account never
displaces or ends their own session. One small, self-contained piece of built-in
account-management functionality — the very code responsible for two-factor setup, password
changes, and profile updates — had simply never been told about that second identity at all. It
had its own fixed idea, from the day it was first wired in, of exactly one place to look for "the
current user," and nothing had ever gone back and taught it about the second, more recently added
way someone could legitimately be using the system.

## Confirmed, not guessed

The instinct here easily could have been "the button's just broken, same shape as the routing bug
from earlier today." It looked similar enough. But a wrong diagnosis here would have led to editing
the wrong file entirely — this needed direct confirmation, not a plausible guess. Checking the
underlying record straight in the database made it unambiguous: the click had genuinely done
something, just not to the account anyone had intended. It had reached out and quietly modified the
helper's own account instead of the one they were standing in for, with literally nothing on screen
to indicate that had happened.

## Choosing not to make it "work"

The tempting fix is always the more impressive one: teach the button to correctly recognize both
kinds of "current user" and make it genuinely work no matter which one is active. That would have
been possible here. It was deliberately not the fix chosen.

Two-factor authentication exists to tie a login to a physical device only its real owner holds.
Someone standing in on another person's behalf scanning that code with their own device would leave
the account's real owner permanently locked out of using two-factor the way it's meant to be used —
tied to *their* device, not someone else's, set up while looking over someone's shoulder rather than
by the person who'll actually rely on it every day after. The safer, and ultimately more correct,
choice was to simply not offer this while standing in for someone else at all — with a plain
explanation in its place, not a mysteriously dead button, and the same rule extended to the two
other actions that share this exact same underlying identity mix-up: changing a password, and
updating profile details.

## A leftover, checked and cleared, not assumed harmless

The one concrete trace of the mix-up — an unconfirmed setup code sitting on the wrong account — got
checked, confirmed harmless (it had never been confirmed, so it had no actual effect on that
account's login), and cleared, only after asking first rather than assuming that was wanted.

Tested (2790/2790).
