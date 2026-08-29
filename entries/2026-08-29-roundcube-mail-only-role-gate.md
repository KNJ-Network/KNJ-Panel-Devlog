# Phase 141 - Roundcube Was Installed on Mail Only, Just Never Wired Up

During readiness testing on a multi-server stack, opening webmail for an account whose mail runs on a linked Mail Only server returned a 403. The webmail application was installed there, but the web server configuration only routed to it for the main role.

The role check was corrected in both the first-install script and the provisioning step so webmail is routed on every role that hosts it, while components that only exist on the main server stay main-only.

In the same pass we added logging to KNJ Webmail's folder-listing failure path, which had been replacing the underlying error with a generic message. The underlying cause could not be fully reproduced because a test server was offline, but the failure is now traceable.

Tested (2506/2506).
