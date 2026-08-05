# Phase 56 - Server Configuration, and What "Done" Actually Means Here

Five items on this section's roadmap. Two were already fully built and just mislabeled. One was a
straightforward build. The last two turned out to be the wrong shape of task, which is worth
explaining because it changes how the rest of the roadmap gets closed out.

Server-wide settings and Timezone configuration were already live on the Server Settings page
(hostname, timezone, upstream DNS resolvers, contact emails, and a graceful reboot button). Neither
had been marked correctly on the public roadmap.

Traffic & visitor statistics was the one real build: a new page per account, linked from List
Accounts, reporting requests, unique visitors, bandwidth, and a response-status breakdown for a
chosen month, plus top pages and referrers. It builds on the parsing the existing bandwidth
tracker already does. Live verification with real test traffic caught a parsing bug before it
shipped, and a verification mistake where the first attempt hit a different site than intended.

The last two, Server role profiles and Multi-server support, already have shipped foundations: a
role setting on every registered server, add and remove management, and DNS clustering for
DNS-only servers. What they lack is role-aware provisioning, meaning a server configured
differently depending on its purpose. That is the same work as the dedicated DNS-only server
build, so a shallow version here would either do nothing real or duplicate that work. Both were
marked honestly on the roadmap as foundation shipped, rather than forcing a checkbox.

That's the admin-side roadmap done as far as this pass goes.
