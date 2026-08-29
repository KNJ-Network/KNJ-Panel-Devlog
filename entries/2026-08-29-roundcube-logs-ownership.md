# Phase 144 - The Directory Roundcube Could Never Write To

The first real use of the new diagnostics page paid off immediately: webmail's log directory had the package's default ownership rather than the one the setup intended, so the webmail process could not write its own logs. The setup step meant to correct this suppressed its own errors, so the failure had been invisible.

We fixed it in two places. The setup script now reports a failure of this step without aborting the install, and the provisioning step re-applies the intended ownership each time it runs, so existing servers repair themselves without manual work.

Whether this alone would resolve the webmail error, or merely let the real error reach a log, was not yet known; the next step was to apply it to a test server and check.

Tested (2506/2506).
