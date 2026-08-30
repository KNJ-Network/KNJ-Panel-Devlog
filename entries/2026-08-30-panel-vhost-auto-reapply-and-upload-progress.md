# Phase 168 - Closing the Loop on the Body-Size Fix

The upload limit fix from Phase 166 had to be applied by hand on the first server, which raised the question of how similar fixes would reach other servers. They would not: a routine update refreshed the provisioning script but did not re-run it.

## Splitting issue and reapply apart

We separated the web-server configuration step from certificate issuance, so the panel update job can regenerate the panel's own configuration after each successful update without contacting the certificate authority. It skips cleanly where no certificate exists yet, and it is isolated from the existing snippet reapply so one failure cannot suppress the other.

## Upload feedback

During a very large upload, the only sign of progress was the browser spinner. Both cPanel import forms now show a progress bar with percentage and byte count.

## Verifying

Confirmed on the development server that the overlay initializes without console errors. Full suite (2571/2571) and `pint --test` before shipping.
