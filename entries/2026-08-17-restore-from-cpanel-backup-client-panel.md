# Phase 90 - Restoring a cPanel Backup Into an Account You Already Have

Yesterday's Restore a cPanel Account shipped one shape of the feature: an admin uploads a real
cPanel backup and it becomes a brand-new account. A user question surfaced a second, genuinely
different shape almost immediately: what if the account already exists, and the customer just
wants their old cPanel-hosted content brought into it?

## Same archive format, a different trust boundary

The two flows share almost everything — the same archive validation, the same `mysql/*.sql`
discovery, the same `homedir/homedir.tar` inspection — but they can't share the extraction step.
The admin-side flow wipes `public_html` first, which is safe because the account it's writing into
didn't exist ten seconds earlier. Doing that to an account a customer is actively using would
delete their live site's own files the moment something in the backup happened to collide.

The account-side variant (`import-cpanel-archive-into-account`, a new provisioning action) extracts
with `--keep-old-files` instead: anything already on disk is left exactly as it is, and only files
genuinely missing from the current site get added. Nothing about "restore" here means "wipe and
replace" — it means "add what's missing," which is the honest, safer reading once the target is
something a real customer already depends on.

## The domain question that caught something else

A user asked what happens if the domain has changed since the backup was taken — does the import
silently break anything? Walking through it surfaced two things:

1. The admin-side flow can't restore under a different domain at all — the new account's domain is
   always read straight from the archive's own DNS zone filename, non-negotiable. So there's no
   mismatch case to handle; it's simply not an operation the feature offers.
2. The public roadmap and in-app copy had been overclaiming since the very first version shipped:
   they said the archive's own DNS records get restored. They don't — only the zone's *filename* was
   ever read, to detect the domain. The account gets a fresh, server-generated zone either way. Not
   a bug in the running code, but a real gap between what was documented and what was true, caught
   from a user's question rather than from testing. Fixed the same session it was found.

The account-side variant doesn't inherit either problem — no domain is created or changed, so
there's genuinely nothing to get wrong there. But it does surface the domain question in a
different form: if the backup's database has the old domain baked into it (a WordPress `siteurl`,
for example), that's still there after import, unrewritten. The results page says so explicitly.

## What's still missing

A real answer to that last gap — a Database Search & Replace tool, so a customer whose domain
changed can find and swap every reference to the old one across their database — is scoped but not
built yet. It's deliberately not folded into this feature, since search-and-replace across
arbitrary tables is useful for any domain change, not just a cPanel-backup restore.

## Next

Database Search & Replace, then back to the two still-open pieces of the original migration-tool
roadmap line: the real Transfer Tool (pulling multiple accounts over SSH from a live cPanel/WHM
server) and Convert Addon Domain to Account.
