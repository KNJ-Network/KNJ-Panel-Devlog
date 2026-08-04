# Phase 49 - API Tokens, and the Security Section Closed

The Security section is done. API Tokens shipped, and external/OAuth login came off the roadmap
rather than staying on it as an unfinished item.

API Tokens already had the easy half built: an admin could mint a token and copy it. What it lacked
was anything for the token to call. The first question was what abilities a token should have, and
the answer was to make each account lifecycle action a distinct, independently grantable ability
rather than one bundled "manage accounts" permission: create, suspend, unsuspend, terminate,
package change, and password reset, alongside read. A token that only needs to suspend accounts
should not also be able to remove them, which the old two-ability list couldn't express.

Building the routes those abilities gate started with a cleanup pass. Account-removal and
password-reset logic lived inside one controller, so both moved to a shared provisioning service,
and the web form and the API now call the same code. DNS record validation, including the checks
that keep malicious record values out of generated zone files, was likewise pulled into a shared
class so the new DNS endpoints reuse it rather than re-implementing a security check.

Every new endpoint checks the token's ability and then the same ownership check every other account
action runs: the ability is a ceiling on what a token can do, not a replacement for confirming it
may touch that account. Live-verified on a development server with real tokens: a read-only token
was refused when trying to suspend, a token with the right abilities walked a real account through
its lifecycle, and an unauthenticated request was refused. The test tokens and account were removed
afterward.

External/OAuth login was removed from the roadmap rather than left marked as in progress, because no
code had been written for it and leaving it there would have misrepresented the state of the
product.
