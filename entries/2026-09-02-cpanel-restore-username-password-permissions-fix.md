# Phase 172 - What a Real-World Restore Actually Breaks

Earlier restore testing used synthetic archives and one real-world backup used to verify the format. A more realistic restore turned up three problems the earlier testing had not.

## Username length

Generated usernames were truncated to seven characters while the original was eight. Because database names derive from the account username, restored databases no longer matched the names inside the site's copied configuration. Usernames are now truncated at eight.

## Database passwords

Restored databases got fresh random passwords, which could never match a configuration file copied from the old host. Like mailbox passwords, database credentials from the backup are now preserved, so the site keeps working without editing its configuration. When none is available, the restore falls back to a new password and rewrites the matching configuration entry. Because the panel does not know the plaintext of a preserved credential, the features that need one, such as one-click phpMyAdmin sign-in, now explain this instead of failing.

## File permissions

Deleting a file in the File Manager failed with a raw 500 because the restored web root had a stricter mode than other provisioning paths. The mode is now consistent, and the delete action reports failures as a readable error.

## What's still a warning

The restore now compares the generated username with the one in the backup and logs a warning, never a block, if they differ.

## Verifying

2673/2673 passing with 7 new tests, plus a live check on the development server.
