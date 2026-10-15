# Phase 262 - The Docs Site That Deployed Itself

The documentation site now runs on the panel it documents, built by the panel's own Deploy from Git, and
every push to it goes live without anyone touching a server.

## What had to change first

Linking a second repository showed that the feature was not ready for it. A repository host accepts a given
deploy key on only one repository, so each link now has its own key: you generate one on the form, add it at
the host, then link, and a link's settings can regenerate it later.

The page was rebuilt around that. Instead of a key box at the top there is a table of current links (where it is
served, the repository and branch, the last deploy and its time), and clicking a row opens that link's settings
in a window: the webhook address, the key, the deploy script, the latest log, the full history and removal.
Links made before the change kept working, so nothing had to be relinked.

Deploying onto an existing subdomain turned up a real bug. A deployment onto a subdomain's own folder was
recording itself against the main site, so deploying or removing it touched the wrong site. Each repository now
remembers which site it serves.

## The guides that all returned 404

Once the docs were live every guide page was missing. The panel was answering a couple of its own paths on
every domain it hosts, so the docs site's pages under one of them never reached it. That is wrong for anyone
who runs a site of their own, so the panel now uses its own ports for those and leaves customer domains alone.
The address people type for webmail keeps working.

## Smaller things the same week

A folder in the File Manager now opens with one click, because the second click of a double-click landed on a
redrawn row and never counted. The slow buttons on Deploy from Git grey out and show a spinner while they work.
The docs site no longer flashes unstyled for a moment as it loads.

Next: more fixes and polish.
