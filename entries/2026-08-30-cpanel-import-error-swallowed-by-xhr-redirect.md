# Phase 169 - The Upload That Vanished

During a large cPanel migration test, the upload finished and the page then came back blank, with no error and no new account.

## What happened

The new upload progress overlay finished the request in the background, and the browser followed the redirect automatically. The page then navigated to the same address a second time, and by then the one-time validation message had already been consumed, so a pristine form was shown.

## The fix

The completion handler now renders the response it already received instead of requesting the page again, so errors and resubmitted values are shown. We verified it with a deliberately invalid archive: the validation message now appears where it should.

## Cleanup

While checking that a failed import leaves nothing behind, we found the cleanup only handled one class of error. It now handles any unexpected failure, matching the other import paths.

Full suite (2572/2572) and `pint --test` before shipping.
