# Phase 57 - Everything Else, and a Dead Setting Found Along the Way

Five items in the roadmap's catch-all section. Three were already built, one was already
correctly scoped, and one, Webhooks, was a genuine gap worth building properly.

DNS resolver configuration, admin notification routing, and remote reboot already lived on the
Server Settings page. Server clustering was accurately marked as partial: DNS clustering is
live, and configuration clustering is its own larger feature.

Webhooks was the real build: an admin can register an endpoint and subscribe it to a small set
of events: account created, suspended, unsuspended, terminated, and SSL issuance failed. Every
delivery is a signed POST, with the delivery outcome tracked on the webhook itself.

Wiring the SSL-failure event surfaced something worth fixing: the SSL-issue notification toggle
had been saved but never read, so it did nothing. Both went in together, and a real email now
goes out through the same notification pipeline broadcast messaging uses.

Live verification took an extra turn. The first delivery attempt against a public echo service
timed out because that service was itself returning errors, confirmed by checking outbound
connectivity directly. A different endpoint succeeded immediately. Then actual account creation
and deletion were triggered through the panel and the webhook fired for both, not just the manual
"Test" button.

That closes every admin roadmap section.
