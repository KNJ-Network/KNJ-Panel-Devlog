# Phase 183 - The Fix That Wasn't the Whole Fix

The dashboard hang from a few hours earlier was supposed to be closed. The fix shipped, tested clean,
went out in a build — and the exact same server still hung on login. Not a regression, not a flaky
retest. The first fix was real and necessary. It just wasn't the only thing broken.

## Ruling out the obvious suspect first

The instinct was to assume the earlier fix hadn't actually taken effect — a stale process, a deploy
that hadn't landed. It had. The version number in the page header proved it. So the honest next step
was to stop assuming and start measuring: call every piece of the dashboard's own render path
directly, one at a time, with real timing. Every single one came back in well under a second. The full
authenticated page, rendered exactly the way a real login would trigger it, came back in under a
second too. The backend had nothing left to be slow about.

## Watching the browser's own words

A detail worth taking literally rather than glossing over: the status bar didn't say the page was
*waiting* for a response. It said it was still *connecting*. That's an earlier, lower-level stage of
loading a page than "the server is thinking" — it means a network connection itself hadn't finished
being established. A fast backend and a browser stuck at "connecting" together point at exactly one
kind of thing: something in the page is trying to reach a destination that isn't actually there to
answer.

## Reading what the page actually asked the browser to fetch

Pulling a real, fully authenticated copy of the page's HTML — logging in for real, over the real
network, through the real reverse proxy, not a shortcut — showed it plainly: every script and
stylesheet tag pointed at a *different port* than the one the page itself had just been served from.
Confirming that port even accepts a connection on this particular server took fifteen seconds to fail.
Not refused, not closed — silently ignored. That specific failure shape only happens one way: a
firewall rule deliberately dropping the attempt instead of rejecting it, which is exactly the correct,
intentional behavior for a port this kind of server was never supposed to have open in the first
place. A browser doesn't know any of that. It just waits.

## The same fix, applied one door too many

The reason that wrong port was baked into every page traces back to a fix from the day before —
solving a real, separate problem: a handful of convenience shortcuts that let a visitor stay on their
own domain while still reaching parts of this panel, which meant those specific requests arrived
carrying someone else's domain name instead of this panel's own. Assets need to know where they
*really* live regardless of what name a request walked in under, and the earlier fix handled that
correctly — for those specific shortcuts. The mistake was scope: it was written as one rule for the
whole server, not one rule for just those doorways. Every other page on the server — the ordinary,
direct, nothing-unusual-about-it admin login — got swept up in the same override, sent looking for its
own assets somewhere they were never going to be for a plain, ordinary visit.

## Deciding it fresh, every time, instead of once for everyone

The honest fix isn't a smarter guess at the right port — it's not answering the question with one
fixed value at all. Every request already carries the one fact that actually distinguishes the two
cases: which name it arrived under. Checking that, freshly, on every single request, and only
redirecting the ones that genuinely came in through one of those special doorways, means an ordinary
request is simply left to find its own assets exactly where it already knows to look. Already-running
servers don't need anyone to fix them by hand either — the check runs early enough to override
whatever old, wrong setting is still sitting there from before, and the next routine update quietly
tidies that old setting away for good.

## Verifying it end to end, not just where it broke

A direct request to the ordinary login page now correctly finds its own assets in place. A request
deliberately sent in under one of the legitimate shortcut names still correctly gets redirected to
where the real assets live. And the specific, narrow case that caused this in the first place — an
ordinary request landing on a server that was never given the port the old rule assumed existed — no
longer gets sent looking for it at all.

Tested (2714/2714).
