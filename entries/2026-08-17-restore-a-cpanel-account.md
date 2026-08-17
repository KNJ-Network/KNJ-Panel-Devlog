# Phase 89 - Restoring a Real cPanel Account, Not a Reimplementation of Our Own

The biggest gap left on the parity list was migration: a way to bring an account off a real
cPanel/WHM server onto KNJ Panel. The panel already had one migration path — export a completed
backup, import it into another KNJ Panel install — but that's a KNJ-to-KNJ format the panel invented
for itself. This is the first feature that has to speak someone else's format.

## What shipped

Admin uploads a real cPanel backup archive (`cpmove-<user>.tar.gz`, or the identically-shaped file a
full WHM backup produces) and it becomes a working account here. Reliably restored: the website's
files (`public_html`), its MySQL databases, and its DNS records. Databases get fresh credentials —
a new database, a new user, a new random password, the same way Database Wizard already creates
one — the old grants are never reused, since a cPanel account's own credentials aren't portable
onto a redesigned login system anyway.

## What's honestly not restored

cPanel's real archive format isn't published anywhere authoritative enough to code every field
against with confidence, and this shipped without a real `cpmove` file to test against — only
cross-referenced third-party write-ups of what the format looks like. So the scope stayed narrow on
purpose: email account passwords, SSL certificates, and cron jobs are all detected in the archive
(the run's results page lists exactly what was found) but never auto-recreated. Guessing at a
format that couldn't be verified risked silently getting something wrong in a way nobody would
notice until a mailbox stopped receiving mail; "found, do this manually" is a worse first
impression but a more honest one. Multi-domain accounts are also out of scope for now — same
"single domain only" line the KNJ-to-KNJ importer already draws, for the same reason.

## The parts that had to be defensive rather than exact

Two things in particular couldn't be pinned down from documentation alone. Where the account's
domain actually lives in the archive turned out to have several plausible answers across different
write-ups (a metadata file, a YAML file, the DNS zone's own filename) — the DNS zone filename is the
one piece every source agreed was reliably named after the domain, so that's the only source this
trusts. And whether the old username shows up as a leading path segment inside `homedir.tar` wasn't
consistent either — so the code inspects the tar's own member listing at runtime to find wherever
`public_html` actually sits, rather than assuming a fixed depth.

## Next

Still open on the same roadmap line: the actual Transfer Tool (pulling multiple accounts over SSH
from a live cPanel/WHM server, not a single-file upload) and Convert Addon Domain to Account —
neither has any existing groundwork to build from. Revisiting the cPanel archive parsing itself
once a real sample archive is available to verify the untested parts against is the more immediate
next step.
