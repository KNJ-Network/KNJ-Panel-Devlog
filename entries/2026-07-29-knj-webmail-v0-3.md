# Phase 25 - Your Own Folders, Filed by Dragging

v0.3.0: users can create their own folders and drag messages onto them to file — the two pieces
missing from feeling like a real mail client rather than a working demo of one. The standard
folders (Inbox, Sent, Drafts, Trash, Spam) stay protected — only a folder a user created can be
renamed or deleted.

Two things only showed up once this was clicked through:

- Creating or deleting a folder failed with an IMAP error, because the library performed an extra
  cleanup step that didn't apply to those operations. That step is switched off now.
- Native browser drag-and-drop can't be driven by automated test tooling, so the move was verified
  by hand, with the underlying endpoint checked directly first.
