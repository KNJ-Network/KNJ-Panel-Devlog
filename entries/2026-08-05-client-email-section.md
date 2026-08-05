# Phase 59 - The Client Email Section, and a Bug the Whole Phase Kept Hiding

Ten items on the account-side Email roadmap. Nine shipped; the tenth is a documented wait on
infrastructure that doesn't exist yet, with no half-built shortcut taken to close it out.

Mailbox Storage Usage, Bulk Address Import, and Delivery Tracking were the straightforward ones:
real disk usage per mailbox, a CSV importer mirroring the webmail's contact importer, and the
existing admin delivery log reader reused with a strict domain check so an account owner only
sees their own mail activity. Mail Filters turned out to be a real gap: per-mailbox filtering
only existed behind Webmail's own login. It is now in the Account Panel directly, with a "global"
option that applies one rule to every mailbox in a single submission.

Spam Scoring and Greylisting opt-out both needed the same discipline: working features that never
reconfigure the shared mail services other customers depend on. An account can ask for something
stricter than the default, never something that changes the default itself.

Mailing Lists needed new infrastructure of its own, kept separate from forwarders so a list
address can never collide with an existing forwarder. It was verified with a real send landing
in two mailboxes from one delivery. Mail Encryption keeps the usual division of responsibility:
the panel generates a certificate, and the account owner downloads a password-protected bundle
and imports it into their own mail client.

Challenge-response needed an honest redesign mid-build. The roadmap wording implied the sender
clicks a confirmation, which wasn't buildable with the available mail tooling. It shipped as
owner-reviewed approval instead: unrecognized senders are held in a Pending folder until the
account owner approves them, verified live with a first message landing in Pending and a second,
after approval, landing in Inbox.

That work also surfaced a real bug: filing rules that pointed at a folder that didn't exist yet
were silently falling back to Inbox. It was fixed and verified by re-running the exact scenario
that exposed it.

Nine features, one bug found and fixed, and one item left correctly marked Planned.
