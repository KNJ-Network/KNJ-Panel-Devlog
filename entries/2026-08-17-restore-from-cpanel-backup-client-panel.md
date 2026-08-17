# Phase 90 - Restoring a cPanel Backup Into an Account You Already Have

The previous phase shipped one shape of the feature: an admin uploads a cPanel backup and it
becomes a brand-new account. A follow-up question surfaced a second, different shape: what if the
account already exists and the customer wants their old content brought into it?

## Same archive format, a different trust boundary

The two flows share the archive validation, database discovery, and home directory inspection, but
can't share the extraction step. The admin-side flow replaces the web root, which is safe because
the account didn't exist moments earlier. Doing that to an account a customer is actively using
could delete their live site's files.

The account-side variant adds only what's missing and leaves anything already on disk exactly as it
is. Nothing about "restore" here means "wipe and replace".

## The domain question

A question about what happens if the domain has changed since the backup was taken led to two
findings:

1. The admin-side flow can't restore under a different domain at all; the new account's domain
   always comes from the archive.
2. The public roadmap and in-app copy said the archive's DNS records get restored, when only the
   zone's name was used to detect the domain, and the account gets a fresh, server-generated zone.
   The documentation was corrected the same day.

The account-side variant doesn't create or change any domain, but if the backup's database has the
old domain baked in, such as a WordPress site URL, it is still there after import. The results page
says so explicitly, and the Database Search & Replace tool addresses it.
