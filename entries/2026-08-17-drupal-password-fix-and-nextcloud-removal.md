# Phase 95 - Finishing What Audit #11 Left Open

Audit #11 deferred three installer password-handling findings (Drupal, Nextcloud, and phpBB)
rather than guess at a fix for tools whose exact behavior hadn't been confirmed. Going back to
finish that research turned up first that phpBB didn't need fixing: it was already handled
properly earlier, with a restricted-permission config file, and the audit's outstanding list was
out of date. It was corrected.

## Drupal: a config file existed, once it was actually looked for

Drush doesn't have wp-cli's prompt convenience, so a first pass assumed no safe option existed. Its
own documentation describes a configuration file holding whatever would otherwise go on the command
line. This was confirmed against the real docs, then implemented like the phpBB fix: a single-use
restricted-permission temp file, deleted whether the install succeeds or fails.

## Nextcloud: a real dead end

The same research for Nextcloud's installer found that its password prompt requires an interactive
terminal, and its file-based alternative is an open upstream feature request. Nextcloud also was
never a great fit for shared hosting, since it wants dedicated resources closer to a VPS workload.
We decided to pull it from the catalog rather than ship it with an unresolved gap or an unverified
workaround. The catalog is now WordPress, Drupal, MediaWiki, and phpBB, and the public and internal
roadmaps were updated to match.

## What's left

The full test suite is green (1413, one fewer because the Nextcloud test was removed), code style is
clean, and the change is deployed and confirmed live. The App Installer's outstanding list from
audit #11 is now empty.
