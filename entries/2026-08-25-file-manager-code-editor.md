# Phase 132 - File Manager Gets a Real Code Editor

The last item of the parallel build alongside Phase 131: the File Manager could list, download, upload,
rename and delete files, but never let anyone see or change what was inside one without downloading,
editing locally and re-uploading. Every file row now has an "Edit" link next to Download, opening a
syntax-highlighted code editor in the browser.

## The one deliberate dependency

This codebase had stayed at zero new JS dependencies through 130-odd phases. Syntax highlighting is
the one place that stance does not hold: recognizing token boundaries across dozens of languages is a
solved problem where reimplementing it is pure risk. CodeMirror 6 is the addition. Language detection
matches the file name and lazy-loads only that language's grammar, which the production build splits
into small chunks.

## What's new

- **Read and write in the File Manager service**, reusing the existing path-resolution boundary every
  other File Manager operation goes through, with a configurable size cap (2 MB default) and rejection
  of binary content.
- **Edit and update actions**: edit renders the editor with the file content embedded; update is a
  JSON `PUT` that the Save button and Ctrl/Cmd+S both call.
- **The editor script**: CodeMirror with a dark theme, a Save keymap, a state indicator
  (clean, dirty, saving, saved, error) and a guard when navigating away with unsaved changes.

## The bug the tests caught before it shipped

The empty-file save test failed because framework middleware turns an empty string into null before
validation, so clearing a file and saving was rejected. The sibling whitespace-trimming middleware has
a sharper edge: it would silently strip a deliberate trailing newline on every save. This is the same
class of bug found in Phase 108 for an internal endpoint, for the same reason: the request carries a
file's exact content, not form input meant to be tidied. Both middlewares now exempt the save
endpoint.

## Verified

17 new or extended tests covering open, save, binary rejection, path traversal and read-only
locations, plus a clean frontend build; full local suite green (1,963 tests) and code-style clean.

Live on the dev server against the deployed build: a disposable PHP file opened with language
auto-detected and highlighting matching the theme's palette, an edit flipped the indicator to unsaved,
saving through the real endpoint wrote the file byte-for-byte including the absence of a trailing
newline, and the test file was removed afterward.
