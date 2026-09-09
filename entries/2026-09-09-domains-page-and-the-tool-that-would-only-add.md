# Phase 213 - The Tool That Would Only Add

A toggle switch is the simplest control there is. Building the one behind it took finding out that the
tool meant to flip it only agreed to go one way.

## What the page needed to say

A domain either forces HTTPS or it doesn't, and until now the panel had no way to show or change that.
The Domains page grew the columns a cPanel install already has: where a domain redirects to, and a
switch for forcing the secure version. It also gained a retry for a stalled certificate and a way in to
DNS. Not every convenience made the cut. Creating an email address and managing DNS zones currently
work only for an account's primary domain, so pointing either at an addon domain's row would have
promised something that silently did the wrong thing. The email shortcut was left out, and the DNS one
appears only on the row where it's true.

## A tool for adding, asked to take away

The certificate tool the panel relies on can turn a redirect on after the fact, but it refuses to turn
one off: it only hardens configurations. A second command looked like a fallback and ran without
complaint, but on a system that already believed the redirect was configured, it did nothing. A page
reporting "done" would have been describing a server that hadn't changed.

The fix stopped asking the tool to change its mind and makes the change directly: the redirect
configuration is removed when the switch is off, and a small dedicated block is written when it is on,
each marked so the next toggle knows what it is looking at. It was verified on a live server by toggling
off and on in a real browser, and then repeating, because once was never enough to call it trustworthy.
