# Phase 93 - Convert Addon Domain to Account

The last item on the original migration-tools roadmap line: promoting an addon domain into its own
top-level hosting account.

## What "ready for it" means

The question before any code was who requests this and where. Real WHM provisions accounts and
leaves charging to a separate system. KNJ Panel makes sure the technical plumbing is available to
external systems through its API. Two requirements followed, neither needing anything new:

- **A cross-account API token was already possible.** The existing admin and reseller API token
  architecture already scopes access by the user who created the token. What was missing was an API
  surface for addon domains at all, with new abilities to gate it.
- **`account.created` needed to fire reliably.** It already existed as a webhook event, but only
  fired from the plain Create Account form, so the import and transfer paths never notified
  anyone. The dispatch moved into the provisioning job itself, firing once an account is genuinely
  Active, so every creation path now emits it.

## Re-parenting instead of exporting and re-importing

The obvious approach, exporting the addon domain and running it through the import pipeline, is the
wrong shape here. Almost everything domain-specific already hangs off the site rather than the
account: DNS zones, mailboxes, forwarders, the catch-all address, Git repos, Quick Sites, and app
installations. Updating the owning account on the existing site record carries all of them over.

Only the files on disk and the account-level setup a fresh account needs (system user, PHP pool,
vhost, database) don't ride along. Databases are the one deliberate gap: databases are
account-scoped, so there's no way to know which belonged to this addon domain. The new account gets
its own fresh default database and nothing else moves automatically.

## The bug only a real conversion would surface

Live-verifying against a real addon domain with an actual file failed immediately: moving a
directory's contents with `mv` doesn't work the way the equivalent copy idiom does. The step now
uses the same copy-then-remove pattern used elsewhere in the provisioning script.

## Live-verified end to end

A real addon domain with a known marker file was converted through the browser UI. Afterward the
marker file was present byte-for-byte in the new account with correct ownership, a fresh database
had been created, and the domain's DNS zone was still attached to the same site record under the
new account, with no DNS work redone.
