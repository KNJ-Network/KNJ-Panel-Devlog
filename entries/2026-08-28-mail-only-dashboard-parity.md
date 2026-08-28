# Phase 138 - The Dashboard Mail Only Never Got

Linking up a Mail Only server and DNS Only servers alongside a Main install surfaced something that had
been missing since Mail Only first shipped: it never got the dedicated dashboard DNS Only got at the
same stage. Logging into a freshly installed Mail Only box fell through to the generic Main dashboard,
with Accounts, Packages and Resellers cards that mean nothing on a box with no sites, plus a red
"Your licence is not currently valid" banner that should not have appeared. Mail Only is licence-exempt
like DNS Only; the banner slipped through because it is embedded in the generic dashboard with no role
check of its own.

This phase builds a real dashboard, not a re-skin. A Mail Only box has no local database of mailboxes:
Main owns every record and pushes flat configuration down to the satellite. So the dashboard's four
cards are genuinely local: Total Mailboxes (counted by a new provisioning action), Mail Queue depth,
Recent Deliveries from the local mail log, and a Service Status panel filtered to the services this
role installs.

The same gap was fixed in the first-run setup wizard: DNS Only gets an inline "Link to Your Main
Server" step, and Mail Only now does too, instead of sending the admin to a separate Server Link page
afterward. The Mail Queue and Mail Delivery Reports pages were also exposed in the Mail Only nav, since
they are diagnostic tools reading the box's own local state.

Full local suite (2480 tests) and the code-style check green. Deployed to the dev server and confirmed
live before the release: nothing broken on the Main dashboard, and the new Mail Only branch renders
correctly under test.
