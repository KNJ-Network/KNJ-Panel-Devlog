# Phase 37 - Backups, and Locking In a Real Path to v1.0

Yesterday ended with a step back rather than a feature: a lot had shipped, but there was never
an actual plan for reaching a releasable state — just whatever seemed like a good idea to build
next. That's fixed now. Four things stand between where the panel is today and a real v1.0:
backups, panel login security (brute-force lockout + 2FA), Cron Jobs, and licensing. Everything
else — however good an idea, however tempting a "quick addition" — is frozen until those four
ship. First up: backups, since a hosting product with no recovery path is the one gap that's
genuinely dangerous to leave unbuilt.

**What a backup actually contains**: the account's files, every one of its databases, all its
mailboxes, and a snapshot of its DNS zone and FTP account config — everything needed to fully
reconstitute the account somewhere else. Written as separate, inspectable pieces per backup
(`files.tar.gz`, one `.sql.gz` per database, a `.tar.gz` per domain's mail, a `.zone` file) rather
than one opaque combined archive — matches how every other multi-part output in this codebase
already works. Local disk only for now; an off-site destination is a deliberate v1.1+ addition,
not an oversight.

**Restore had to be real, not a stub**, or none of this was worth building. Both the admin (any
account, disaster recovery) and the account owner (their own account, "I broke something") get a
genuine restore that regenerates files, databases, and mail from exactly what a specific backup
captured — not the account's current state, which may have drifted since.

**Ownership was the actual design problem here**, more than the mechanics of tar and mysqldump. A
backup has to be readable and deletable by the panel app itself, but never by the hosting
account's own website process — a compromised site must not be able to read its own backup
history, let alone tamper with or delete it. That meant `root:knjpanel` ownership throughout, group-
writable so the app can prune and delete without a privileged round-trip for every single
deletion. Getting the *chain* of that right took two real fixes caught only by testing the actual
delete button, not just the write path: the ancestor directories `mkdir -p` creates on the way to
a new backup weren't inheriting the same group-write access, so the app could create and read
backups but never delete one — and a backup that failed partway through left its partial output
stuck `root:root`, permanently unreachable by the app, briefly world-readable besides. Both fixed:
the ancestor chain gets its ownership set explicitly, and a cleanup trap now removes a failed
backup's output entirely rather than leaving broken data behind.

**Live-verified for real, not just against a unit test**, because "did tar and mysqldump run
without error" isn't the same claim as "does a restore actually put things back." Created a
disposable test account on the real dev server, backed it up, deliberately corrupted a file and a
database row afterward, restored, and confirmed both came back byte-for-byte identical — file
content, database content, ownership, all of it. That same live pass caught a bug no unit test
could have: every account also gets a default database named exactly its own username with no
suffix, and the manifest validation only accepted the suffixed shape, so a real account's default
database failed to back up every time. Fixed and re-verified before shipping.

Also found, and deliberately left alone for now: removing an account doesn't drop its MySQL
databases — they're orphaned, not deleted. Real gap, but unrelated to backups, so it's flagged as
its own follow-up rather than folded in here.
