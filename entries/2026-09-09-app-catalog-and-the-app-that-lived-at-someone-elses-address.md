# Phase 212 - The App That Lived at Someone Else's Address

Four apps in a catalog is a list. Growing it into something worth calling a catalog meant picking the
next app and letting it teach us what the first four had gotten away with not needing.

## Tabs first

Every installer now declares which category it belongs to, such as CMS & Blogs, Forums & Community, or
Wikis & Docs, and the catalog page builds its filter tabs from that: All Apps, then one per category in
use. Nothing talks to the server twice; switching tabs only decides which cards stay visible, the same
approach as the sidebar search box.

## An app that had never needed a home file

The first four apps each have a real login page that exists as a file. Bookstack doesn't. Every page is
a route decided by code when the request arrives. The server's handling of requests that don't match a
real file was written with file-based apps in mind, so an installed route-based app could end up
answered by the wrong site.

The fix gives every installed app its own address that only it answers to, kept current whenever an app
is installed or removed.

## The address that agreed with itself, twice

Two smaller versions of the same lesson turned up only by actually loading the page rather than trusting
that "installed successfully" meant "reachable." Bookstack builds its asset URLs from one address it is
told at setup. If that address is slightly wrong, such as the wrong scheme, a browser refuses to fetch
assets over an insecure link on a secure page, and the page renders as little more than an outline. It is
now given the one address that is actually correct.

None of this shipped on the strength of "the install finished." It shipped once a browser could load
the real login screen and a real admin account could sign in.
