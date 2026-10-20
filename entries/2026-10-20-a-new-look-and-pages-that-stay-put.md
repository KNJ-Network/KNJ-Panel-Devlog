# Phase 267 - A New Look, and Pages That Stay Put

The support desk gave the panel a lot of new screens, and next to them the older ones looked tired. So
the whole panel got a fresh coat of paint, and while testing it we found two things worth fixing properly.

## The new look

Both areas now have a dark header and sidebar. The controller dashboard opens with a Server overview
banner that puts accounts, anything needing attention, packages, resellers and server health in one
glance, and the username and Log out moved into the thin bar along the top. The customer dashboard has a
banner of its own with the domain, plan, disk and bandwidth use and server details, a search box that
finds any tool by name, and a tidy grid of everything the plan includes, with anything not included
greyed out.

## Text that was never quite clear

Small text looked soft on some computers. The cause was simple: the panel asked for a particular
typeface but never actually shipped it, so every browser fell back to whatever its system happened to
have, and some of those render tiny text badly. The panel now ships its own fonts, picking Segoe UI where
the computer has it and an open-licence look-alike where it does not, and the small text on the dark
banners got a little more weight. Same sizes, much easier to read.

## The page that would not sit still

To see how the panel behaves at the size of a real hosting company, a test server was filled with a few
hundred demo accounts and several hundred tickets. The account list started flickering. Pages that wait
for something in the background, such as an account being set up, used to reload themselves every few
seconds until it finished, and when something never finished the page reloaded for ever. Eleven pages
worked that way.

They now update only the part that changed. A status pill moves from Issuing to Active and a log gains
new lines, while typed text, ticked boxes and scroll position are left alone. Updating stops when
nothing is pending, after a quarter of an hour at most, when the tab is in the background, and when the
person is signed out.

The new look shipped in 0.21.0. The in-place updating is finished and in the development build, ready
for the next release.
