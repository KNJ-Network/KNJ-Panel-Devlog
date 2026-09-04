# Phase 194 - Standing In Someone Else's Shoes, Correctly

The routing fix from earlier the same day worked. Double-checking it by using the support access mode,
which lets staff see exactly what a customer sees without needing their password, turned up something
else: clicking the same button appeared to do nothing. No error, no QR code.

## Two ways of being someone else

A system that lets one person temporarily see the world through another's account has to answer, on
every request, who "the current user" is. Get it right everywhere and the mode is seamless. Get it wrong
in one place and an action lands on the wrong account instead of failing loudly, which is worse than an
obvious error.

The system had this right almost everywhere. One small piece of built-in account-management
functionality, covering two-factor setup, password changes and profile updates, had simply never been
told about the second identity.

## Confirmed, not guessed

It would have been easy to assume this was the same shape as the routing bug from earlier. A wrong
diagnosis would have meant editing the wrong file, so it was confirmed directly: the click had done
something, just not to the intended account.

## Choosing not to make it "work"

The tempting fix was to make the button work in both modes. It was deliberately not chosen. Two-factor
authentication exists to tie a login to a device only the real owner holds. Setting it up on someone
else's behalf would defeat that purpose. So these three actions are now not offered while in support
access mode, with a plain explanation in place of a dead button.

## A leftover, checked and cleared

The one trace of the mix-up, an unconfirmed setup code on the wrong account, was confirmed harmless
(it had never been confirmed, so it had no effect on login) and cleared.

Tested (2790/2790).
