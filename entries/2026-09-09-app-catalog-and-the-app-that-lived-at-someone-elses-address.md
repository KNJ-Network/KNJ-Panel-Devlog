# Phase 212 - The App That Lived at Someone Else's Address

Four apps in a catalog is a list. Growing it into something worth calling a catalog meant picking
the next app and letting it teach us what the first four had quietly gotten away with not needing.

## Tabs first

Every driver now says which shelf it belongs on — CMS & Blogs, Forums & Community, Wikis & Docs,
whatever comes next — and the catalog page reads that off to build its own filter tabs: All Apps,
then one per shelf actually in use. Nothing here talks to the server twice. Every card was already
sitting on the one page; switching tabs just decides which of them stay visible, the same instinct
already living in this codebase's sidebar search box, aimed at a new shelf.

## An app that had never needed a home file

The four apps already in the catalog all have one thing in common that's easy to miss until it's
gone: somewhere on disk, at a real, literal path, sits the page an admin logs in through.
WordPress's is `wp-login.php`. You can point a web server at it directly and it just works, the
same as pointing it at any other file that happens to exist.

Bookstack doesn't have one. Nothing in it does. Every page — the homepage, the login screen, a
single book's contents — is a *route*, decided by code the moment the request arrives, not a file
waiting to be found. And the web server serving every account on this box had one rule for handling
a request that doesn't match a real file: hand it to whatever sits at the site's own front door.

For four apps that never needed that rule to be smarter, nobody noticed it was also wrong. Install
Bookstack into a subfolder, ask for its login page, and the server would dutifully search for a file
that couldn't exist, give up exactly as designed, and hand the request to the site's own homepage
instead — which had never heard of Bookstack, and said so, politely and unhelpfully, in its own
words. Confirmed the hard way: diff the two responses byte for byte, and they're identical. Two
different addresses, one answer, and it belonged to neither of them.

The fix teaches the server something the first four apps never had to ask for: give every installed
app its own address, one that only it answers to, kept current the instant anything's installed or
removed. Get that wrong and the cost isn't a broken link — it's every visitor to that address quietly
being handed someone else's front door and never being told.

## The address that agreed with itself, twice

Two smaller versions of the same lesson, both found only by actually loading the page rather than
trusting that "installed successfully" meant "reachable."

Bookstack doesn't just link to its own pages — it *is* its own pages, generated from one address it
was told about at setup and trusted from then on, baked into its own stylesheets and scripts, not
just its own links. Tell it the wrong address — off by one folder, or the wrong `http`/`https` — and
it doesn't 404 gracefully. It builds every asset URL from the wrong premise, and a browser that
already trusts the *real* page over a locked connection flatly refuses to fetch anything served
insecurely alongside it, no error dialog, no partial rendering — just a page that loaded and rendered
nothing but its own outline. Fixed by telling it the one address that was actually true the whole
time: not almost right, not off by a folder, not the wrong lock — just correct, checked against
what's actually already sitting on the box before ever answering.

None of this shipped on the strength of "the install finished." It shipped once a browser could load
the real login screen and a real admin account could sign into it — the same bar as saying yes to a
question, not the same as never having doubted it.
