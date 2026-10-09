# Phase 246 - The Bug Hiding Behind the Workaround

The first fix for a bug is often not the last one. This is the story of a workaround that failed
for a reason nobody had found yet.

## A connection test that never connected

Pointing a hosting panel at a brand-new backup server should be a five-minute job: enter the
details, press Validate. Instead the test failed instantly with a complaint that an object key was
empty — an error that comes from the storage library itself, before any request is ever sent. The
server was fine, the credentials were fine; the panel was failing before it spoke to anyone.

Cause: before every test or upload, the panel prepared the destination by "creating its root
folder". That step exists for SFTP and FTP, whose adapters never create the configured folder
themselves. Cloud object storage has no folders at all — with no path prefix configured, "create
the root folder" turned into "store an object with no name", which the library rightly refuses.
A destination at the top level of a bucket could never validate, never upload.

## The workaround that didn't work

The obvious workaround was to set a path prefix, so the folder step had a real name to create. Tried
live — same error. Reading the stored settings on the server showed why: the prefix field was empty
in the database even though it had been typed into the form.

The add-destination form is one form with three sections (cloud storage, SFTP, FTP), and only the
selected type is shown. But hidden fields are still submitted, and all three sections used the same
field names. The browser sent the path prefix three times — the typed value, then two empty ones —
and the server keeps the last. The prefix was silently erased on every save.

## What shipped

Both bugs, together, because neither could be seen until the other was fixed. The folder step is
skipped for cloud storage. The form now disables every section that isn't selected, so only the
chosen type's values are ever sent. Verified in a real browser against the old and new page script:
the old one submitted the prefix as three values with the typed one lost; the new one submits
exactly one. The same hint text now says plainly what the prefix is and that leaving it blank is
fine.
