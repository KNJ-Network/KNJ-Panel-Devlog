# Phase 37 - Backups, and Locking In a Real Path to v1.0

Yesterday ended with a step back rather than a feature: a lot had shipped, and it was time to
prioritize what a releasable state needs. A hosting product with no recovery path is a real gap, so
backups came first.

**What a backup actually contains**: the account's files, every one of its databases, all its
mailboxes, and a snapshot of its DNS zone and FTP account config — everything needed to
reconstitute the account somewhere else. Each backup is written as separate, inspectable pieces
rather than one opaque archive. Backups are stored locally for now.

**Restore had to be real, not a stub**, or none of this was worth building. Both the admin (any
account, disaster recovery) and the account owner (their own account, "I broke something") get a
genuine restore that regenerates files, databases, and mail from exactly what a specific backup
captured — not the account's current state, which may have drifted since.

**Ownership was the actual design problem**, more than the mechanics of archiving. A backup has to
be readable and deletable by the panel app but not by the hosting account's own website process, so
a compromised site can't read or tamper with its own backup history. Testing the actual delete
button, not just the write path, turned up two permission-chain bugs: the app could create and read
backups but not delete them, and a failed backup could leave partial output with the wrong
ownership. Both were fixed, and failed backups are now cleaned up entirely.

**Live-verified**, because "did the backup run without error" isn't the same claim as "does a
restore actually put things back." On a development server, a disposable test account was backed up,
a file and a database row were deliberately corrupted, and a restore brought both back identical.
That same pass caught a bug no unit test could have: an account's default database name didn't
match what the backup validation expected. Fixed and re-verified before shipping.
