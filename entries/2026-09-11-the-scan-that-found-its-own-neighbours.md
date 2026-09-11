# Phase 224 - The Scan That Found Its Own Neighbours

Yesterday's discover feature was built on a fair assumption: a domain and its own
subdirectories are one self-contained thing, and scanning them tells you what's really
installed there. Testing it against a real account instead of a fixture found the seam in
that assumption within minutes.

## A folder that belongs to someone else

Subdomains don't get their own separate patch of disk by default — they live one level
inside the domain they were created under. `blog.example.com` and `example.com` aren't two
unrelated places; `blog.example.com`'s files sit in a folder literally named `blog`, inside
`example.com`'s own document root. Two different sites, sharing a directory tree, connected
by nothing more than a subfolder name.

A scan that walks a domain's own folder plus its immediate subfolders doesn't know that.
It just sees a subfolder with a working WordPress install in it and reports it — technically
true, and completely misleading. That folder isn't an unregistered subdirectory install
belonging to the parent domain. It's a whole other site's own front door, one that already
has its own record, its own document root, its own place in the account. The scan wasn't
wrong about what it found. It was wrong about whose it was.

## Why a fixture wouldn't have caught it

The original tests all passed. They tested exactly what they were told to test: a real
install turns up, an already-tracked one doesn't turn up twice. What they never modeled was
two real sites sharing the same corner of the filesystem — because building that scenario by
hand requires already knowing it's the thing to build, and the whole reason it went
unnoticed is that it doesn't look like a corner case from the code. It looks like the normal
case. Most accounts have exactly this shape: one main domain, one or two subdomains tucked
inside it.

Real data didn't just fail to confirm the bug — it revealed a version of it far messier
than any test would have reached for. The genuine article among the mess: one install with
no site record pointing at it at all, no domain, nothing — reachable only as a bare folder
under someone else's document root, precisely the shape a restored backup leaves behind.
That's the case discovery exists for. Sitting right next to it, wrongly caught in the same
net, were two other folders that weren't orphans at all — they already belonged to someone.

## Telling "unclaimed" from "already spoken for"

The fix isn't a smarter scan. It's a smaller one — check what's already registered elsewhere
on the account, and if a found folder is actually a sibling site's own home, leave it out.
Not because it's fake. Because claiming it here would mean claiming it in the wrong place —
attaching a real, working, already-addressed site to the wrong owner's record, just because
it happened to be sitting nearby.

## What actually finished

The scan now only offers what's genuinely unclaimed on the domain it was asked to check.
A neighbour's own front door doesn't get mistaken for an open door of your own, no matter
how close together they sit on disk.
