# Phase 160 - Three Things the Real Servers Wanted

Feedback from running a multi-server stack led to three changes shipped together.

## phpMyAdmin: installed everywhere except where it mattered

phpMyAdmin worked on the development server but was missing on a main server that had been set up before it was added to the install flow. We added an idempotent check to the provisioning step so already-provisioned servers install it automatically the next time a hostname is re-confirmed.

## No more surprise databases

Account creation used to create a database for every account, mirroring cPanel's default. We decided account owners should choose to add a database, so the auto-create was removed from account creation and addon-domain conversion. Import paths still import existing databases.

## Update all linked servers from one page

Rolling out an update meant visiting each server in turn. Linked servers now report their panel version and Manage Servers shows when each is current. A new bulk button starts each server's normal update and shows progress in one place, with a clear timeout message if a server stalls. The button only appears when there is a linked server.

Tested (2532/2532).
