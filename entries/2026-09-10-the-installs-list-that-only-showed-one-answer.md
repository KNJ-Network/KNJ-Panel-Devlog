# Phase 223 - The Installs List That Only Showed One Answer

The root-install fix landed clean. A real WordPress site went in at a real subdomain's own
document root, no subfolder needed, exactly as asked. The feedback that came back a few minutes
later wasn't about that working — it was about what the page did once it had.

## A filter where a filter didn't belong

The installs table sat under a domain picker, and the table only ever showed whichever
domain the picker currently pointed at. That made sense when the picker's only job was
choosing where a new install would go — but the table wasn't the install form. It was a
list of things that already existed, each one already labeled with its own domain right there
in its own row. Filtering it by the picker meant the picker was doing two jobs at once, and
only one of them made any sense: find the thing you're looking for by switching away from
it first.

The fix wasn't cleverness, it was noticing the list didn't need the filter it had been given.
Every row already said which domain it belonged to. Once that's true, showing all of them at
once isn't a bigger feature — it's removing a restriction that never earned its keep. The
picker's second job — deciding where a *new* install lands — is the only job it needed to be
doing.

## The gap between "installed" and "known about"

The second half of the ask was a different shape of problem. A real WordPress install existed
— files on disk, a working site, actual visitors — but the panel had no row for it, because it
arrived by a different door: a cPanel backup, restored wholesale, WordPress and all, with
nothing about the restore process ever telling this panel's own install tracker it had just
gained a tenant. From the panel's point of view, that install simply doesn't exist. Not
broken, not failed — invisible.

That's a genuinely different bug shape than "read the wrong domain." Nothing is wrong; the
thing the code is supposed to notice just never happened where the code was watching. The
only honest fix is to go looking on purpose — scan the place an install would live, check
whether something real is actually running there, and offer to catch the tracker up.

"Something real is actually running there" turned out to be worth being strict about. A
config file sitting in a directory isn't proof of a working install — it's proof someone once
started one, or it's a leftover, or it's half a restore. The only signal worth trusting is the
same tool that already manages every tracked install successfully reporting back a real
version number. If that tool can't even load the site, the panel has no business offering to
adopt it either.

## Adoption, not installation

Once a candidate clears that bar, registering it isn't the same operation as installing one.
A fresh install provisions a database, a database user, an admin account — real infrastructure
built from nothing. A discovered install already has all of that, wired up its own way,
working. Registering it should touch none of that. It's a single row, describing something
that's already true, added one confirmed candidate at a time rather than a bulk sweep — because
a false positive here doesn't just clutter a list, it starts a panel "managing," and later
possibly deleting, a site nobody asked it to.

## What actually finished

The installs list now tells you everything on the account, because every row already knew how
to say which domain it was talking about. The domain picker went back to doing the one job it
was always meant to have. And a site that arrived through the back door can be found, checked
against the same standard as everything installed the normal way, and brought into the list on
purpose — one confirmed answer at a time, never a guess.
