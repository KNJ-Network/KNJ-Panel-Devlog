# Phase 182 - What Else Were We Not Looking For

Finding the missing contacts wasn't the end of that question, it was the start of a bigger one: if a
real customer's archive was quietly missing one whole category of data, what else might a *different*
customer's archive carry that this restore feature has never looked for? Not a hypothetical — this
restore feature was never going to be used by exactly one person, exactly once.

## Reading the whole thing, on purpose

The honest way to answer that question is to actually look — every top-level entry in a real,
~975MB archive, cross-referenced one at a time against what the restore feature actually reads from
versus what it extracts and quietly ignores. Most of what turned up was either already a known,
documented limitation (SSL certificates, cron jobs used to be one of these) or genuinely empty for
this particular account — no custom email forwarders beyond cPanel's own defaults, no extra FTP
logins beyond the account's own built-in one. Three categories stood out differently: real,
already-existing panel features with a real file format sitting in the archive, and zero restore code
looking at any of it.

## Building on what already existed, not from scratch

The encouraging discovery, once the gap was clear: none of the three needed new privileged
infrastructure. Email forwarders, FTP accounts, and cron jobs already have complete create/restore
paths in this codebase — built for the account owner's own UI, and for KNJ's own account backup and
restore feature. The FTP path in particular already had a real hash-transplant restore method,
proven in production by the backup feature, just never called from anywhere in the cPanel importer.
This was a parsing-and-wiring problem, not a build-a-new-subsystem problem.

Each format needed its own small judgment call about what counts as "real" data worth restoring
versus boilerplate the source system writes into every account whether a customer asked for it or
not. Email forwarding's own default catch-all rule — "unmatched mail for this domain, deliver it
locally" — isn't a forward at all, and importing it as one would create a confusing, pointless extra
row. The FTP account list always carries an internal, automatically-created account cPanel uses
purely to let a website's own stats page peek at its access logs; a customer never sees it, never
uses it, and it must never show up disguised as something they configured.

## The password problem, again

FTP accounts hit the exact same shape of problem the mailbox restore already solved once: every new
account gets an FTP login auto-created the moment the account itself is created, with a brand-new
random password that has nothing to do with the old one. Naively restoring the archive's own FTP
account by its old name would leave two working logins side by side — the fresh one nobody told the
customer about, and the old one with the password they actually remember. The fix mirrors the
mailbox fix precisely: recognize the archive's "this is the account's own primary login" entry by
matching its home directory to the account's own root, and transplant the original password hash
onto the row that already exists, through the real restore path — so the actual, working, system-level
FTP password changes, not just what a database column happens to say.

## Verifying it end to end, not just in isolation

Every parser has its own test built from the real, live-confirmed line format for that data type —
including one deliberately annoying case, a forwarder configured to send mail to more than one
address at once, since this panel's own forwarder record can only ever hold one destination and the
overflow has to be reported, never silently dropped. Beyond the parsers themselves, a full restore run
through the real, unmodified entry point proves the wiring actually fires — not a mocked call, a
generated cron job and forwarder landing in the real database with the real log lines an admin would
actually see.

Tested (2709/2709).
