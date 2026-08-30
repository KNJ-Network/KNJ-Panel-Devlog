# Phase 162 - The Save Button Nobody Knew They Had to Click

Shipping a proxy change surfaced a smaller issue: the generated web-server snippet for account sites was only rewritten when an admin saved Nginx Settings. Upgrading the panel did not regenerate it, so existing installs kept the old configuration until someone clicked Save & Apply.

## The fix

The panel update job now re-applies the existing Nginx Settings after a successful update, in a way that can log a failure but never turn a successful update into a failed one.

## Proving it

On the first update that introduces this step, it could not run, because the worker had loaded the previous code before the update replaced it. From each server's next update onward it applies. Five dedicated tests cover the success path, failure handling and the failed-update case. All servers were updated and the site was verified.

Tested (2546/2546).
