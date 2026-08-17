# Phase 89 - Restoring a Real cPanel Account, Not a Reimplementation of Our Own

The biggest gap left on the parity list was migration: a way to bring an account off a cPanel/WHM
server onto KNJ Panel. The panel already had one migration path, exporting a backup and importing it
into another KNJ Panel install, but that is a format the panel invented for itself. This is the
first feature that has to speak someone else's format.

## What shipped

An admin uploads a cPanel backup archive and it becomes a working account here. Reliably restored:
the website's files, its MySQL databases, and its DNS zone domain. Databases get fresh credentials,
a new database, user, and random password, the same way Database Wizard creates one, since a cPanel
account's credentials aren't portable onto a different login system anyway.

## What's honestly not restored

cPanel's archive format isn't published authoritatively enough to code every field against with
confidence, so the scope stayed narrow on purpose. Email account passwords, SSL certificates, and
cron jobs are detected in the archive and listed on the run's results page, but never
auto-recreated. "Found, do this manually" is a worse first impression but a more honest one than
guessing wrong silently. Multi-domain accounts are out of scope for now, the same single-domain
line the KNJ-to-KNJ importer draws.

## The parts that had to be defensive rather than exact

Where the account's domain lives in the archive has several plausible answers across different
references, so the code trusts only the source every reference agreed on. And the layout of the
home directory archive varies, so the code inspects the archive's own listing at runtime rather
than assuming a fixed structure.
