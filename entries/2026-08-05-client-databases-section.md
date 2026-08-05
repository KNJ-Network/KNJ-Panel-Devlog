# Phase 62 - The Client Databases Section, a CI Blind Spot, and a Real Remote Connection

Three items, plus a CI check that was fixed along the way.

The guided database wizard was the simple one: a "Quick Setup" section that creates a database,
a user, and a full-access grant in one submission, rolling back whatever it already created if a
later step fails.

The visual database browser needed a real security decision, not just a UI. Browsing tables
could reuse the existing read-only browser, but query execution could not reuse the admin tool,
which has server-wide reach. Instead, submitted queries authenticate as one of the account's own
database users, so the database's own permission system is the isolation boundary. This was
verified live: the user's grants were scoped to exactly one database.

Remote database access needed more than a database change. Checking the server first showed the
feature was impossible at the network level until the right pieces were in place. It was built so
that each allowed host gets its own separate database account, remote listening is only enabled
while at least one remote host exists, and firewall rules are managed automatically. It was
verified with a real client on a different machine connecting over the internet, and then
confirmed blocked again once the host was removed.

The other event this phase: GitHub Actions had been failing on a code-style check while the test
suite passed. Fixed with one formatting run and no behavioral changes, and CI status is now
checked after every push.

Databases section closed out.
