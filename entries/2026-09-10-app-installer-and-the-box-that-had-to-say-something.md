# Phase 217 - The Box That Had to Say Something

A subdomain exists to be its own address. That's the whole point of making one — not a folder living
under another site's roof, but its own front door, with its own document root behind it. Someone
who goes to the trouble of creating `blog.example.com` on purpose, rather than just using
`example.com/blog`, is telling you something about where they want things to live.

The install form didn't ask where. It asked what to call the folder.

## A field that assumed its own answer

Every text box on that form meant what it said, except one. Site title, admin email, admin
password — each maps straight onto something real, no interpretation required. The subdirectory box
looked the same as the rest, sat in the same list, asked with the same quiet insistence: fill this
in before you can continue. But it was answering a question nobody had actually asked yet — not
"what do you want to call this," but "where do you want this" — and it had already decided, on
everyone's behalf, that the answer was always going to be a folder.

For most installs that's exactly right, and nobody notices the assumption because it never gets
tested. It only shows up the moment someone builds a subdomain specifically so an app can live at
its own root, fills out every other field correctly, and then hits a box that won't let them leave
without inventing a folder name they don't want and don't need.

## What the blank was actually for

The box wasn't missing a feature. It was missing permission to be empty. Nothing about the box
itself, or the database it wrote to, or even the code reading it back out afterward, assumed a
folder had to exist — the model already knew how to describe an app living at a site's own root, had
known for a while, with nothing pointing at it. The requirement lived one layer up: a validation
rule insisting on a value, and a second one underneath it, in the part of the system that actually
touches disk, insisting the same thing independently. Two separate places had each, on their own,
decided a blank answer wasn't a real answer.

Neither one was wrong to check. A path that might be empty has to be handled deliberately everywhere
it's used — the directory to write into, the address to install at, the config line telling the
finished app what to call itself — and skipping that work is exactly how "leave it blank" quietly
turns into "write into the wrong place, or the site's own home directory itself, by accident." The
fix wasn't removing a check. It was teaching every place downstream what an empty answer actually
means, so that removing the requirement upstream stopped being dangerous.

That mattered most in exactly one place: the part of the system that cleans up after itself. Taking
an app back out of a folder means deleting the folder. Taking an app back out of a site's own front
door does not — that door has to stay standing for the site itself. Get that distinction wrong and
"remove this app" quietly means "remove the folder everything else in the site was counting on."
Caught before it ever ran anywhere real, but it's the kind of mistake that only announces itself once,
loudly, on someone else's server.

## The box, now honest

The field still asks. It just no longer insists. Leave it blank and the install lands exactly where
the address itself already pointed — no invented folder, no manual cleanup afterward, no explaining
to a subdomain why it isn't being allowed to be its own front door. The placeholder text got quieter
too, closer to a suggestion than an answer already filled in, and the sentence above it stopped
promising a subfolder as if that were the only shape an install could take.

Small box. It just had to be allowed to mean nothing, on purpose, when nothing was the right answer.
