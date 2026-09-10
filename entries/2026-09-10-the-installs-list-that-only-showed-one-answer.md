# Phase 223 - The Installs List That Only Showed One Answer

The root-install fix landed cleanly: a WordPress site went in at a subdomain's own document root with no
subfolder. Feedback on what the page did afterwards was about something else.

## A filter where a filter didn't belong

The installs table sat under a domain picker and only showed the domain the picker pointed at. That made
sense when the picker's only job was choosing where a new install would go, but the table is a list of
things that already exist, each labeled with its own domain. Filtering it by the picker meant the picker
was doing two jobs, and only one made sense. The table now shows everything, and the picker only decides
where a new install lands.

## The gap between "installed" and "known about"

The second half was a different problem. A real WordPress install can exist on disk with no row in the
panel, for example when it arrives via a restored cPanel backup that never told the install tracker
about it. From the panel's point of view it is invisible, not broken.

The only honest fix is to look on purpose: scan where an install would live, check whether something
real is running there, and offer to catch the tracker up. "Something real" is held to a strict standard:
a leftover config file isn't proof, and the panel only offers to adopt a site if the same tooling that
manages tracked installs can load it and report a real version.

## Adoption, not installation

Registering a discovered install isn't the same as installing one. It touches none of the existing
infrastructure and adds a single row describing what is already true, one confirmed candidate at a time
rather than in a bulk sweep, because a false positive would start the panel managing, and possibly later
deleting, a site nobody asked it to.

## What actually finished

The installs list shows everything on the account, the picker does its one job, and a site that arrived
another way can be found, checked and brought into the list on purpose.
