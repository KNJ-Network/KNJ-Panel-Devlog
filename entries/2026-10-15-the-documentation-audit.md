# Phase 265 - The Documentation Audit

Documentation drifts. Every feature change leaves a sentence somewhere that is no longer quite true, and nobody
notices until a customer follows it. So after the security work we checked every guide against the real product.

## How we checked

Two checks, run separately. The first read each change shipped since the last audit and compared the guide that
covers it with what the code actually does: the buttons, the messages, the limits. The second started from the
other end, with the product's own menus, tabs and routes, and asked whether each one has a page.

## What turned up

Mostly small things, but real ones. The File Manager guide said you are always asked before an upload replaces a
file, when files dropped onto another folder replace without asking. The malware guide named the old quarantine
folder. Clicking a folder's name opens it now, and the guide still said to double-click. A backup guide promised an
unreachable destination is given up on in seconds, which is true for two of the three kinds and not the third.

Three guides were missing altogether: KNJ Webmail, the panel's own mail client, which had no page of its own;
signing in and recovering access; and the Server Link page that DNS and mail servers use to connect to their main
server. They are written now, from the code rather than from memory.

## Two real bugs, found by writing about them

Documenting how a feature behaves is a good way to notice it behaves wrongly. Deleting a subscriber did not take
them off the linked mailing list until someone pressed Sync now. And KNJ Webmail
left off an attachment over 20 MB and sent the message without it. Both are fixed: removal now follows through, and an oversize attachment stops the send with a clear message.

Next: keep the guides in step with the product.
