# Phase 95 - Finishing What Audit #11 Left Open

Audit #11 explicitly deferred three installer password-exposure findings — Drupal, Nextcloud, and
phpBB — rather than guess at a fix for CLI tools whose exact stdin/config behavior hadn't been
confirmed. Going back to actually finish that research turned up something worth noting first: phpBB
didn't need fixing at all. It was already done, properly, earlier in the same session — a chmod-600
YAML config file, not a bare flag — and the audit report's own "still outstanding" list was simply
wrong about it, written without re-checking the live code first. Corrected on the spot.

## Drupal: a config file existed, once it was actually looked for

Drush doesn't have wp-cli's `--prompt=` convenience, and the instinct on a first pass was to assume
that meant no safe option existed. It wasn't that simple — drush's own configuration documentation
describes exactly the shape this needed: a `--config=<file>` pointing at a YAML file with
`command: site: install: options:` holding whatever would otherwise go on the command line,
`db-url` and `account-pass` included. Confirmed against the real docs rather than assumed from a
half-remembered pattern, then implemented the same way phpBB's existing fix already works: a
single-use temp file, chmod 600, deleted whether the install succeeds or fails — reusing the same
`set -e`-safe cleanup pattern already established elsewhere in this script rather than writing a new
one.

## Nextcloud: a real dead end, not a skipped one

The same research approach for Nextcloud's `occ maintenance:install` landed somewhere genuinely
different. Its password prompt isn't just "no stdin support" — reading its actual command source
confirmed the hidden-input mechanism explicitly disables its own fallback
(`setHiddenFallback(false)`), meaning it requires a real pseudo-terminal to work at all; piping a
plain value in wouldn't do anything. Its own file-based config option — the thing that would have
closed this cleanly — turned out to be a real, still-open GitHub feature request from 2018, never
implemented. Eight years and counting.

That, paired with a separate observation that Nextcloud was never a great fit for shared hosting to
begin with — it wants dedicated RAM, real storage headroom, and ideally its own cron worker, closer
to a VPS workload than a typical hosting account — settled the decision: pull it from the catalog
rather than ship it with a known-open password-exposure gap or bolt on an unverified
pseudo-terminal workaround just to keep the slot filled. Driver class, provisioning script action,
and its test all came out together; `docs/roadmap.md` and the public roadmap now describe the
current four-app catalog (WordPress, Drupal, MediaWiki, phpBB) instead of the five that used to be
there.

## What's left

Full test suite still green (1413, one fewer than before — the removed Nextcloud test, nothing
else moved), `pint` clean, deployed and confirmed live. The App Installer's outstanding items list
in audit #11 is now actually empty rather than carrying three deferred entries forward.

## Next

Continuing down the gap list.
