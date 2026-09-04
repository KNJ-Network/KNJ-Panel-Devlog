# Phase 193 - The List That Needed One More Name

A tester's whole job, done well, is being the person who tries the thing nobody else got around
to yet. That's exactly what turned this one up: not a code review, not an automated check, just
someone genuinely trying to turn on Two-Factor Authentication from their own account, the way a
real customer eventually would, and hitting a wall doing it.

## A door with a guest list

A page built to double as two different things — a customer's own address acting as a doorway
straight into their account — has to make a judgment call on every single request that arrives:
does this belong to the account's own regular content, or is it one of a small set of special pages
that have to be reached exactly as typed, no doorway dressing added? Login is one of those. Logging
out is another. And it turns out, so is every screen involved in actually managing an account's own
security settings — turning on two-factor codes, confirming a password before a sensitive change,
updating that password once confirmed.

That list of "don't touch these, let them through as-is" had already been built once, the hard way,
after login itself turned up broken. But a list built by finding problems one at a time only
protects against the ones already found. It had never occurred to consider the account's own
security-settings pages — nobody had tried using them through this particular doorway yet, so
nothing about them had ever been wrong in an observable way. The list looked complete. It wasn't.

## The exact same shape, spotted twice in the same day

This is now the third time in one day that this exact pattern has shown up: a name for a page,
never explicitly added to a short guest list, quietly turned into the same "not found" that a
completely different kind of mistake produces. Same failure, three separate causes, three separate
short lists that each needed one more entry. That's not a coincidence worth ignoring — it's a sign
that the approach itself, correct as it is, has a cost: every new kind of top-level page this
system ever grows has to remember to add itself to this list, and nothing forces that to happen
automatically. Worth watching for a fourth time before deciding whether the approach needs to
change shape entirely, rather than just growing longer.

## Fixing the one found, and the one nobody had found yet

The immediate fix mirrors the earlier ones exactly: add the missing name to the list, in both
places a customer's own domain can reach the account area. And because the exact same shape of gap
existed in a second, related place — the admin side's own equivalent doorway, which had never had
any exceptions carved out of it at all — that one got the same treatment at the same time, before
anyone had to go find it the hard way too. Nobody had reported that one yet. It was going to be
found eventually, the same way this one was; better to close it now than wait for the next person
who happens to try the right thing at the right moment.

Every already-set-up account picks this up on its own, the same self-repairing check already built
for the earlier fixes, extended once more — nothing to redo by hand, nowhere it can be missed.

## Letting someone try to break it

The most useful thing about having a second, genuinely curious person poking at every corner of
this isn't that they're more thorough than automated testing — it's that they try things in the
order a real, unscripted person actually would, driven by their own curiosity rather than a
checklist someone else wrote in advance. A checklist only ever tests what someone already thought
to write down. A person who likes trying everything finds the gaps between the items on it.

Tested (2780/2780).
