# Phase 101 - Deploy from Git Rebuild, and a Real Subdomains Feature to Build It On

The goal was to rebuild Deploy from Git so it works with private repositories and per-account
deploy keys, the way mature panels handle it, then ship it, cut a release, update the test server,
and test it for real.

## Comparing against the real thing first

Before writing code, the existing feature was compared with how other panels handle repository
linking. Three real functional gaps appeared, not just polish:

1. **HTTPS-only, no private repos.** There was no way to pull a private repository.
2. **No post-pull build step.** A deploy was a plain update with no way to run a build afterward.
3. **Branch was free-text.** There was no way to see which branches exist before typing one.

## A prerequisite nobody expected: no subdomains at all

The plan assumed the create form's domain picker could reuse the App Installer's mechanism, but the
installer has no domain choice at all, and the panel treated every non-primary site as an
addon/parked domain with no real subdomain concept. A genuine Subdomains feature is distinct and
cheap to support: a folder inside the parent's web root, one new record in the existing DNS zone,
and its own certificate. So Subdomains was built first, as a standalone feature.

## What shipped, in build order

1. **Real Subdomains**: a subdomain service, a parent-site link on each site record, and
   provisioning actions to add and remove them. SSL reuses the issuance job every site already uses.
   Removal deliberately leaves the folder in place and only tears down the web configuration.
2. **SSH deploy keys**: one reusable key pair per hosting account, stored encrypted and only
   written out temporarily for a single operation.
3. **SSH-capable clone and fetch, plus branch listing**: the clone and pull actions detect HTTPS or
   SSH from the repository URL. Branch listing is read-only and needs no elevated rights.
4. **Custom post-deploy build scripts**: an optional script per repository, run as the account's own
   user after a deploy. A failing build now fails the deploy run instead of reporting success.
5. **Rebuilt create form**: deploy onto the main domain, or create and deploy onto a new subdomain
   in one step, with a live path preview and a branch picker fetched from the real remote.

## Two bugs found only by testing live

- **A failed clone left an orphaned subdomain behind**, blocking a retry under the same name. The
  controller now removes the just-created subdomain if the clone fails.
- **Removing a subdomain leaves its folder on disk by design**, which then blocks reusing the same
  name. That isn't a bug, but it cost a few minutes of confusion, so it's written down here.

## Live verification on the development server

Testing used the development server's real test account, with real DNS and a real certificate,
in a real browser:

- A deploy key was generated and its public key displayed.
- Branch fetch against a public GitHub repository found all branches and defaulted correctly.
- An HTTPS clone onto the main domain with a custom deploy script confirmed on disk that the script
  ran as the account's own user.
- A full main-domain-to-new-subdomain deploy: the subdomain was created, DNS resolved, a per-host
  certificate was issued with the subdomain's own name, and files landed in the correct nested path.
- The orphaned-subdomain bug was found, fixed, and re-verified.

## Shipped and tested for real

The full local suite stayed green throughout (1509 tests by the end) and code-style checks were
clean on every commit. The release was cut, synced to the release repository, and installed on the
test server through its own "Update Now" button, proving the whole upgrade mechanism carries the
change correctly. Migrations for both new columns ran cleanly on that real upgrade.

One gap was left open rather than silently skipped: a live SSH clone of a private repository wasn't
tested end to end, because it needs a key added to an external repository's settings. The SSH path
is covered by automated tests and mirrors the already-proven transfer feature's pattern, and a real
private-repository test is worth doing before calling this fully closed.
