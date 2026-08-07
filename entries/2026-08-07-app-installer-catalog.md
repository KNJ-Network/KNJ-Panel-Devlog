# Phase 69 - The App Installer Becomes a Real Catalog, and phpBB Takes Some Work

The App Installer had done one thing: install WordPress synchronously, blocking the request for up
to two minutes. This phase turns it into a catalog. Drupal, Nextcloud, MediaWiki, and phpBB join
WordPress, each through its own official non-interactive installer, and the install flow moves
into a queued job with a live log, the same pattern Package Manager uses.

## The shape of it

A small driver interface replaced the WordPress-only logic. The WordPress driver is that same
logic behind the interface, regression-tested to behave identically, including its update-check
and staging-clone features, which stay WordPress-only on purpose. A controller records each
installation as it starts, so duplicate installs at the same path are still rejected, and a job
streams the installer's output into a log the browser polls.

Drupal, Nextcloud, and MediaWiki went in cleanly, each with one small bug caught by running a real
install. Generated database passwords are now alphanumeric-only because one installer doesn't
reliably decode symbols. Nextcloud's large archive legitimately contains internal links, so the
archive safety check was narrowed from rejecting all links to validating where each one points,
and the install timeout was raised once real timing showed no headroom. MediaWiki's release server
rejects requests with no identifying header, an easy fix.

## phpBB, or: three ways to be wrong about a real installer

phpBB looked like the same shape of work and was the most stubborn. Three failures, each only
visible from a real install and each traced back to upstream source:

**The version was wrong.** The download location we had planned around is frozen at an old release,
and every patch since is published only as a tagged source release. The driver now resolves the
latest tag and builds from it the way phpBB's contributor documentation describes.

**The wrong installer entrypoint.** The command first chosen expects an already-installed
application, which a fresh install doesn't have. phpBB ships a separate, dedicated entrypoint for
exactly that situation.

**The config file's shape was wrong.** The installer's configuration parser expects a list of
config sets rather than a single set, so a flat file was misread. The fix was a single leading
list marker.

Building from the current release also resolved a compatibility issue with the server's newer PHP
version. The final run completed all 13 installer steps, the forum served correctly, and cleanup
left nothing behind.

Every one of these fixes shipped only after a real install failed with it, was diagnosed from the
real error and upstream source, and was re-verified with another real install.

The catalog is done: five apps, each real, each async with a live log.
