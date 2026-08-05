# Phase 57 - Everything Else, and a Dead Setting Found Along the Way

Five items in the roadmap's catch-all section. Three were already built, one was already correctly
scoped, and one — Webhooks — was a genuine gap worth building properly rather than as a checkbox.

DNS resolver configuration, admin notification routing, and remote reboot all turned out to already
live on the Server Settings page, shipped back in July alongside timezone and hostname config. Same
story as a few sections back: the public roadmap hadn't caught up with what the code already did.
Server clustering was already accurately marked as partial — DNS clustering genuinely live,
configuration clustering (account/package sync across servers) correctly deferred as its own larger
future feature.

Webhooks was the real build: an admin can register an endpoint and subscribe it to a small, fixed
set of events that actually mean something in this codebase — account created, suspended,
unsuspended, terminated, and SSL issuance failed — rather than trying to expose every internal state
change. Every delivery is a signed POST (HMAC-SHA256 over the raw body, verifiable via an
`X-KNJ-Signature` header), with the delivery outcome tracked directly on the webhook row rather than
a separate log table — enough to answer "is this actually working" without building a whole delivery
history viewer for a v1.

Wiring the SSL-failure event surfaced something worth fixing on its own: `notify_on_ssl_issues`, a
setting Server Settings has saved since July, was never actually read anywhere — the toggle existed,
did nothing. Same fix effort as the webhook event itself, so both went in together: a real email now
goes out through the same notification pipeline broadcast messaging already uses.

Live verification took an extra turn worth noting. The first delivery attempt, against httpbin.org,
timed out — worth chasing down properly rather than assuming the code was wrong. Turned out to be
httpbin.org's own load balancer returning 503s at the time, confirmed by a direct `curl` from the
server showing outbound connectivity was completely fine. Switched to a different echo endpoint,
and the same delivery succeeded immediately. Then the real test: triggered actual account creation
and deletion through the panel and watched the webhook fire for both, not just the manual "Test"
button — the difference between "the feature works" and "the feature is wired into anything real."

That closes every item across every admin roadmap section. Next: a full pass back through all of it,
section by section, to make sure everything marked Live still genuinely is — the same kind of gap
that turned up repeatedly this pass shouldn't be trusted to have stopped just because the roadmap
now says 100%.
