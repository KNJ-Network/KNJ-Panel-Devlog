# Phase 132 - File Manager Gets a Real Code Editor

The last item on the parallel build alongside Phase 131's alerting work: File Manager could list,
download, upload, rename, and delete files, but never let anyone see or change what was *inside*
one without downloading it, editing locally, and re-uploading. Every file row now has an "Edit"
link next to Download, opening a full syntax-highlighted code editor in the browser.

## The one deliberate dependency

This codebase has stayed at zero new JS dependencies through 130-odd phases — webmail's rich
compose uses a sanitized contenteditable, every live-polling feature is hand-rolled `fetch()`, the
admin dashboard's charts are inline SVG. Real syntax highlighting is the one place that stance
doesn't hold: recognizing correct token boundaries across a few dozen languages is exactly the kind
of solved problem where reimplementing it is pure risk with no payoff. CodeMirror 6 is the
addition — `codemirror`, `@codemirror/language-data`, `@codemirror/theme-one-dark`, plus the
`@codemirror/{state,view,commands,language}` packages it already pulls in transitively, now listed
explicitly since `resources/js/file-editor.js` imports from them directly.

Language detection uses `@codemirror/language-data`'s `LanguageDescription.matchFilename()` against
the opened file's name, then lazy-loads only that one language's grammar via its own `.load()` —
confirmed in the production Vite build, which split every supported language into its own small
chunk (PHP, Python, YAML, Dockerfile, dozens more) rather than bundling all of them into the page
that opens a single `.php` file.

## What's new

- `FileManagerService::read()` / `write()` — reuse the existing `resolvePath()` security boundary
  (the same containment check every other File Manager operation already goes through), add a
  configurable size cap (`filemanager.editor_max_bytes`, 2 MB default — plenty for real source
  files, keeps the whole thing in memory for CodeMirror rather than inventing a chunked-editing
  story), and reject binary content via the same null-byte-in-the-first-chunk heuristic git itself
  uses to decide whether to diff a file as text.
- `FileManagerController::edit()` / `update()` — `edit()` renders the editor page with the file's
  content embedded directly (no separate read round-trip); `update()` is a JSON `PUT` endpoint the
  editor's Save button and Ctrl/Cmd+S both call, matching the existing `{error: message}` / 422
  convention `GitDeployController` already uses for its own JSON action.
- `resources/js/file-editor.js` — mounts CodeMirror with the oneDark theme, a Save keymap, a dirty-
  state indicator (clean/dirty/saving/saved/error, each with its own color), and a `beforeunload`
  guard so navigating away with unsaved changes prompts first.

## The bug the tests caught before it shipped

`FileEditorControllerTest::test_save_allows_an_empty_file` failed with a 422 the first time it ran:
"the content field must be a string." The cause was Laravel's own default
`ConvertEmptyStringsToNull` middleware — an empty save request's `content` becomes `null` before
validation ever sees it, so "the user cleared the whole file and hit save" turned into a rejected
request instead of a real zero-byte write. The sibling middleware, `TrimStrings`, has the sharper
edge: it silently strips leading/trailing whitespace from *every* save, meaning a deliberate
trailing newline (something almost every text-file convention expects) would vanish on save with no
error at all — the exact same class of bug Phase 108 found and fixed for the Mail Only dispatch
endpoint, for the same underlying reason: this request carries a file's exact byte content, not
form input meant to be tidied. Fixed the same way — both middlewares now exempt `account/files/edit`
by request path, with a comment pointing at the precedent so the next person doesn't have to
rediscover the reasoning from scratch.

## Verified

17 new/extended tests (8 `FileManagerService::read()`/`write()` cases in
`FileManagerServiceTest`, 7 in the new `FileEditorControllerTest` covering open/save/binary-
rejection/path-traversal/read-only-location, both via `RefreshDatabase` sandboxes matching every
other File Manager test's shape) plus a clean `npm run build`. Full local suite green (1,963 tests)
and `pint --dirty` clean before deploy.

Live on `panel-dev`, against the real deployed build (not a local re-run): logged in as the
account-side test user, opened a real disposable `.php` file with a function and string
interpolation, confirmed the language badge auto-detected "PHP" and the token colors in the actual
rendered DOM matched One Dark's real palette (keywords purple, strings green, variables pink,
function calls blue — read directly off computed styles, not assumed). Edited the file through a
real DOM text-insertion (the editor's own dirty-tracking flipped to "Unsaved changes" correctly),
saved through the real `PUT` endpoint, and confirmed byte-for-byte on disk over SSH — including that
the edit's lack of a trailing newline was preserved exactly, proving the middleware fix actually
holds in production and not just in the test sandbox. Disposable test file removed afterward.
