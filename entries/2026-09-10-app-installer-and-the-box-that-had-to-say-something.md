# Phase 217 - The Box That Had to Say Something

A subdomain exists to be its own address: not a folder under another site's roof, but its own front door
with its own document root. Someone who creates `blog.example.com` on purpose, rather than using
`example.com/blog`, is telling you something about where they want things to live.

The install form didn't ask where. It asked what to call the folder.

## A field that assumed its own answer

Every text box on that form meant what it said, except one. Site title, admin email and admin password
each map straight onto something real. The subdirectory box looked the same and was required, but it was
really answering "where do you want this" and had already decided the answer was always a folder.

For most installs that's right, so the assumption never gets tested. It only shows when someone builds a
subdomain so an app can live at its own root, fills in everything else, and then can't proceed without
inventing a folder name they don't want.

## What the blank was actually for

The box wasn't missing a feature; it was missing permission to be empty. The underlying model already
knew how to describe an app at a site's own root. The requirement lived in two separate validation
layers, each independently deciding a blank answer wasn't a real answer.

Neither check was wrong to exist. An empty path has to be handled deliberately everywhere it is used,
or "leave it blank" turns into "write into the wrong place by accident". The fix wasn't removing a
check; it was teaching every place downstream what an empty answer means.

That mattered most for cleanup. Removing an app from a folder means deleting the folder, but removing an
app from a site's own root must not, because the site itself depends on it. This was caught before it
ran anywhere real.

## The box, now honest

The field still asks but no longer insists. Leave it blank and the install lands where the address
already pointed. The placeholder text is quieter, and the sentence above it no longer promises a
subfolder as the only possibility.

Small box. It just had to be allowed to mean nothing, on purpose, when nothing was the right answer.
