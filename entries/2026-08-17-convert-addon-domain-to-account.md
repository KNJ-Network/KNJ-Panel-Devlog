# Phase 93 - Convert Addon Domain to Account

The last item on the original migration-tools roadmap line: promoting an addon domain into its
own top-level hosting account, with an eye toward a billing system that doesn't exist yet.

## The billing question, and what "ready for it" actually means

Before writing any code, the real question was: who requests this, and where? Real WHM has the
same split cPanel-adjacent tooling always has — WHM provisions accounts, a separate billing system
(WHMCS, in the usual case) decides what gets charged and when. KNJ Panel doesn't sell hosting
itself and shouldn't start pretending to; it should just make sure the plumbing exists for a future
billing system to see what a customer has and trigger the technical side once something's paid for.

That resolved into two concrete requirements, neither of which needed inventing anything new:

- **A privileged, cross-account API token was already possible.** The existing admin/reseller
  Sanctum token architecture (`Whm\ApiTokenController`) already lets a token see and act across
  every account under a reseller, or the whole server for an admin — because the scope check is
  tied to the *user* minting the token, not to the token itself. What was actually missing was
  narrower: an API surface for addon domains at all, and new `domains:read`/`domains:convert`
  abilities to gate it.
- **`account.created` needed to fire reliably.** It already existed as a webhook event, but only
  fired from the plain "Create Account" form — cPanel import, generic import, and the Transfer
  Tool all created real accounts through the same `createAccount()` call without ever notifying
  anyone. Moved the dispatch into `ProvisionAccountJob` itself (fires once an account is genuinely
  Active, not just requested) so every account-creation path gets it uniformly, this one included.

## Re-parenting instead of exporting and re-importing

The obvious first instinct — export the addon domain's content, then run it through the same
import pipeline Restore a cPanel Account and Transfer Tool both use — turns out to be the wrong
shape for this particular job. `Site.account_id` is a plain foreign key, and almost everything
domain-specific in this codebase already hangs off `site_id` rather than `account_id`: DNS zones,
mailboxes, mail forwarders, the catch-all address, Git repos, Quick Sites, app installations. A
single `UPDATE sites SET account_id = ...` on the *existing* row carries every one of those over
for free — no re-creation, no risk of a domain-uniqueness constraint colliding with itself.

Only two things don't ride along with that update: the files on disk (a real `mv`... well, `cp -a`
and `rm`, see below) and the account-level plumbing a fresh account always needs — a system user,
a PHP-FPM pool, a vhost, a database. Databases are the one deliberate gap: there's no way to know
which of the old account's other databases (if any) belonged to this specific addon domain —
databases are account-scoped everywhere in this codebase, and in real cPanel too. Same cut the
cPanel-import and account-migration features already made for the analogous problem; the new
account gets its own fresh default database and nothing else moves automatically.

## The bug only a real conversion would surface

Live-verifying this against a real addon domain with an actual file in it failed immediately:

```
mv: cannot move '/home/old/domains/example.com/public_html/.' to '/home/new/public_html/.': Device or resource busy
```

`cp -a "dir/."  "dest/"` is a completely standard idiom for "copy the contents of this directory,
not the directory itself" — used twice elsewhere in this same provisioning script. `mv` doesn't
actually support that idiom the same way, even though it looks like it should: the kernel refuses
to rename a directory's own `.` entry, so every real conversion failed the instant it tried to move
files. Switched to the same `cp -a` + `rm` pattern the script already uses for the phpBB and
WordPress staging-clone install actions, rather than reaching for something new.

## Live-verified end to end

Real addon domain, a real file with a known marker string, converted through the actual browser UI.
Confirmed afterward on the server: the marker file present byte-for-byte in the new account's home
directory with correct ownership, a fresh database created, and — the part worth calling out — the
addon domain's DNS zone still pointing at the exact same `site_id`, now living under the new
account, with no DNS work re-done at all.

## Next

Nothing left open on the original migration-tools line. Next up: the admin/controller dashboard —
filling it out with real stats, monitors, and quick actions to match how full the client dashboard
already looks, per the user's own request.
