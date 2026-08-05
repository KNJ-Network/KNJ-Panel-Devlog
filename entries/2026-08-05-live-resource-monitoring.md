# Phase 55 - Live Resource Monitoring, and Backup & Server Status Closed Out

One item left in this section: a real process list and disk usage view, with the ability to stop a
runaway process. The closest existing pattern was the database process list built for Database
Maintenance Tools, but this needed to work at the operating-system level across every process.

Listing needed no special privilege and was confirmed against the live server before building.
Stopping a process is the one part that needs elevated rights, and the one part worth being
careful with. The kill action validates its target against the live system at the moment of the
request and refuses to stop processes the panel and hosted sites depend on, pointing the admin at
Service Manager's restart button instead.

Live verification was the real test: a disposable process was started on the server, stopped
through the panel, and confirmed gone. Then the same button was used on a protected service. It
was refused with the expected message and the site kept serving requests throughout. That is the
check that matters here: proving the guardrail holds, not just that the happy path works.
