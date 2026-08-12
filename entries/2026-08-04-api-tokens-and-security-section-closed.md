# Phase 49 - API Tokens, and the Security Section Closed

The Security section is done. API Tokens shipped this session, and external/OAuth login came off
the roadmap entirely rather than staying on it as an unfinished promise — more on why below.

API Tokens already had the easy half built: an admin could mint a Sanctum token and copy it. What
it lacked was anything for that token to actually call. Before writing a single route, the question
worth answering was what abilities a token should even have — and the answer came from thinking
through how a billing integration would actually need to drive the account lifecycle, since this
API doubles as the foundation a future billing system will automate against: create, suspend,
unsuspend, remove, change package, reset password. Each of those needs to be a distinct,
independently grantable ability, not one bundled "manage accounts" permission. A billing
integration token that only ever needs to suspend an account on non-payment should never also be
able to terminate one, and the old two-ability list (`accounts:read` / `accounts:write`) had no way
to make that distinction. It's now seven: read, create, suspend, unsuspend, terminate, package,
password — one per real lifecycle action a billing system would actually need to trigger.

Building the routes those abilities gate turned into a small cleanup pass first. The account-removal
logic and the password-reset logic both lived as private methods on the Controller area's own
`AccountController`,
which meant the new API controller would've had to duplicate them. Both moved onto
`AccountProvisioningService` instead, where the account-creation logic already lived from an earlier
session — so the web form and the API now call the exact same code for every account lifecycle
action, not two copies that could quietly drift apart. The DNS record validation — including the
newline exclusion that's the actual thing stopping a malicious TXT or SRV value from injecting extra
lines into a generated zone file — got the same treatment, pulled out of the account-side
`DnsController` into a small shared class so the new DNS API endpoints reuse it exactly rather than
re-implementing a security check.

Every new endpoint checks the token's ability with `tokenCan()` and then the same ownership check
every other account action already runs — the ability is a ceiling on what a token can do, never a
replacement for confirming it's allowed to touch that account at all. Live-verified against
panel-dev with real tokens: a read-only token correctly got a 403 trying to suspend an account, a
token with create+suspend+terminate abilities walked a real account through creation, an attempted
suspend from the wrong token, and cleanup, and an unauthenticated request got a flat 401. Both test
tokens and the throwaway account were removed afterward.

External/OAuth login is off the roadmap rather than marked "planned" or "in progress." Google Cloud
Console's billing verification wouldn't cooperate no matter how many times it was retried, and
leaving the item sitting there implying the product itself wasn't finished — when the actual holdup
was a third party's signup flow — wasn't an honest way to represent it. No code was ever written for
it, so there was nothing to revert; it'll come back when it's worth the friction, not before.
