# Phase 25 - Your Own Folders, Filed by Dragging

v0.3.0: users can create their own folders and drag messages onto them to file — the two
pieces genuinely missing from feeling like a real mail client rather than a working demo of
one. The standard folders (Inbox, Sent, Drafts, Trash, Spam) stay protected — only something a
user created themselves can be renamed or deleted.

Two bugs only showed up once this was actually clicked through, not in review:

- Creating or deleting a folder was failing outright with "BAD No mailbox selected." The IMAP
  library runs an EXPUNGE right after either operation by default — which needs a mailbox
  already selected, and a brand-new connection has nothing selected yet. Expunging has nothing
  to do with creating or deleting a folder in the first place, so that's switched off now.
- Native browser drag-and-drop turned out to be untestable by browser automation entirely —
  not a bug in the feature, just a real limit of synthetic input events, which can't trigger
  the OS-level drag gesture a real browser requires. Verified the underlying move by hand
  instead, with the endpoint itself checked directly first.

Next up is the bigger one: Outlook-style rules that file incoming mail into a folder
automatically based on conditions, plus a settings page to manage those rules and the
existing per-mailbox autoresponder. The mail server already has everything that needs —
Sieve filtering is installed and already runs autoresponders today, so this isn't new
infrastructure, just a new use of infrastructure that's already there.
