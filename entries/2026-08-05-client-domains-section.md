# Phase 60 - The Client Domains Section, and Two Bugs Older Than This Build

Three items on the account-side Domains roadmap. All three shipped, and along the way turned up
two bugs unrelated to the features being built: one in a mail-filter fix from the previous
phase, the other in database provisioning.

Dynamic DNS is the straightforward one: each host gets its own A record and an unguessable
update URL, since a router or cron job polling on an interval can't carry an Authorization
header the way a browser can. It was verified with a real DNS query after a real update
request, and the common case of polling with an unchanged IP skips the zone rewrite entirely.

Quick Site Publisher needed a design decision before any code: the existing "Website Coming Soon"
placeholder is rewritten every time an account or domain is provisioned, so anything written
there another way would eventually be overwritten. It was built as its own template instead. Live
testing on the first publish showed one call was unnecessary, and it was removed rather than
worked around.

WordPress management closed out the App Installer's oldest open item: check for updates, apply
them, and clone a working copy to a staging path. Live verification showed the original
installer had never been proven against a real WordPress site on the development server, so the
tooling was added to the server bootstrap with checksum verification. That unblocked a second,
older bug: a generated database password containing a backslash was stored as a different
password than the one written to the site's configuration, breaking the connection with no
error. It was reproduced, fixed by escaping the value before it reaches the database, and
confirmed with a round trip.

Three features, two bugs, found by live verification rather than the test suite alone. Domains
section closed out.
