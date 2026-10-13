# Phase 257 - The File the Panel Was Not Allowed to Touch

A customer's own bug report arrived through the person who runs the server: uploads into some folders did
nothing, the code editor opened blank, and saving any file said "Could not open the file". It read like one
big failure. It was three small ones.

## Made by someone else

The File Manager works with the panel's own limited permissions. That works for anything the panel created.
It does not work for files and folders the account made with its own tools: over SFTP, from a git checkout,
or restored from a backup. The panel had always needed that access and never asked for it.

Now a refused write asks a small, deliberately narrow helper for access to exactly that file or folder, and
tries again. The helper refuses anything it should not touch. It was checked on the development server with a
real account against deliberately hostile paths, and refused all of them.

## A warning that was swallowed

One detail cost an hour. The first version decided "permission denied" by asking PHP for its last error. In
the framework the panel uses, a suppressed warning is handled and never recorded there, so the question
always came back empty and the new code never ran. It now asks the file system directly.

## The blank editor

When the account panel is opened through a customer's own domain, the editor's script is fetched from the
panel's own hostname, and the browser needs the server to say that is allowed. It had not been said, so the
editor stayed blank. One header fixes it, applied on upgrade to servers that already exist.

## Smaller things

Uploading a name that already exists now replaces the file; renaming onto an existing name and failing to
create a folder now say so in plain words.
