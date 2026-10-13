# Phase 259 - The Fix That Passed Everywhere but Production

The customer's File Manager problem was fixed twice before it was actually fixed. Both times the tests passed.
The reason is worth writing down.

## Three places it worked

The fix gave the panel a way to ask for write access to one file or folder an account's own tools had made.
It worked when run from a command line on the server. It worked in the automated tests. It worked on a local
copy of the panel. And then, on the real development server, through the real web page, uploading into a
subfolder failed with exactly the message the customer had reported.

## The environment nobody was testing

The panel's web server runs with tighter restrictions than a shell does, and those restrictions blocked one
thing the helper needed. From a shell, or locally, nothing got in the way. From a web request, it was
refused, the failure was logged where nobody was looking, and the customer saw the same generic message as
before.

The remedy had already been used once for another feature that hit the same wall: run the helper through a
separate, narrowly scoped mechanism. The failure is now also written to the log with the helper's own words.

## What changed in how it is tested

Everything was then run again through the real pages on the real server, as an account whose files were made
the way the customer's were: uploads, replacing a file, renames, new folders, archives, permissions, the
Trash, both editors. All passed. The lesson is that anything that depends on the web server's restrictions
has to be tested through a real web request on a real server. Commands, unit tests and a local copy cannot
see that difference.
