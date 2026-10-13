# Phase 258 - The File Manager That Learned to Forgive

After a customer's bug report was fixed, we decided to go beyond a fix: stop patching the File Manager and build the one it should have been, with a recycle bin that has a "skip the trash" option and a proper HTML editor.

## One screen instead of a table of links

The old page was a list with a few links on each row. The new one is a real file browser: a folder tree that
loads as you open it, a toolbar, a sortable list with size, date, type and permissions, a right-click menu,
keyboard shortcuts, and drag and drop in both directions. Files and whole folders can be dragged in from the
desktop, and items can be dragged onto a folder to move them. It is plain JavaScript talking to a small set
of JSON endpoints, and every name from the server goes onto the page as text, never as markup.

## A recycle bin, and why it needed a helper

Deleting now moves things into a Trash in the account's own space and keeps them for thirty days. The delete window offers one checkbox to skip it. Restoring never overwrites something that has taken the old place, and it recreates a folder that has since gone. A daily job clears out what has outstayed its thirty days.

## Archives are where care matters

Compress and extract are the delicate part of any file manager, because an archive is untrusted input. They run with the customer's own permissions and extraction is deliberately conservative. It was tested on the development server against deliberately malformed archives; each bad entry was refused, the good files inside still came out, and nothing outside the account was touched.

## An editor for pages, not just code

The HTML editor shows the page as visitors will see it, with its own styles, inside a frame that cannot run
the page's scripts. Beside it sits the source in a code editor, a split view, and a preview with phone and
tablet widths. Links, pictures from your own files, tables, find and replace, and a recovered draft if the tab
closes early. The one honest caveat is that a visual editor rewrites the HTML in its own tidy form, so
anyone who cares about the exact spacing of their code is pointed at the source view.

## What testing in a real browser found

Three small bugs only showed up on screen: a dialog button that stayed disabled after a restore, the editor's
own helper style leaking into saved pages, and the word "null" in the path bar. And a stray permission change on the development server briefly stopped automatic deployment there; it was fixed.
