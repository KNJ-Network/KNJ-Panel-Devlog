# Phase 166 - The 4GB Limit That Was Never Real

A migration test with a large cPanel backup failed partway through the upload, even though the feature advertises a 4GB limit.

## The cause

The panel's own web service had no explicit request-size setting, so it fell back to a small default, and the PHP upload limits were still at stock values until an admin saved them. The PHP settings form also capped values below the advertised limit. The queued import jobs were already scoped with long timeouts, so the extraction step was unaffected.

## The fix

We set request-size and timeout limits for the panel's own service in both the install path and the update path, and raised the ceiling on the PHP settings form. The defaults for hosting accounts were left unchanged; only the artificial cap was removed. Existing servers pick the change up the next time an admin saves Server Setup.

## Verifying

Full suite (2563/2563) and `pint --test` first. The test archive itself was confirmed intact, so the failure was purely configuration. The migration is retried after the normal release process.
