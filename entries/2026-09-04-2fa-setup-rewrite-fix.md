# Phase 193 - The List That Needed One More Name

A tester's job, done well, is trying the thing nobody else got around to yet. That's what turned this
one up: not a code review or an automated check, just someone trying to turn on Two-Factor
Authentication from their own account, the way a real customer eventually would, and hitting a wall.

## A door with a guest list

A customer's own domain can act as a doorway into their account area, and every request through it has
to be classified: is it the site's regular content, or one of a small set of special pages that must be
reached exactly as typed? Login is one. Logging out is another. It turns out that the account's
security-settings pages are too.

That list of exceptions had been built up as problems were found, which means it only protects against
the ones already found. Nobody had tried the security-settings pages through this path yet, so the
list looked complete. It wasn't.

## The same shape again

This was the third instance of the same pattern in a short period: a page name missing from a short
exceptions list, surfacing as a "not found" page. The approach is sound, but it has a maintenance cost,
since every new kind of top-level page has to be added to the list. That is worth keeping in mind when
adding new pages.

## Fixing the one found, and its sibling

The fix adds the missing names in both places a customer's domain can reach the account area. A
related gap on the admin side was closed at the same time, before it was reported. Already set-up
accounts pick the change up automatically, with nothing to redo by hand.

## Letting someone try to break it

The most useful thing about a second, curious tester isn't that they're more thorough than automated
testing. They try things in the order a real person would, rather than following a checklist someone
wrote in advance. A checklist only tests what someone already thought to write down.

Tested (2780/2780).
