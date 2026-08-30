# Phase 167 - The Save That Ate Its Own Response

Saving PHP Settings in WHM wrote the configuration correctly but returned a 502 to the browser, and the page then showed stale values.

## Cause

The save reloaded PHP-FPM synchronously, which restarted the worker handling the admin's own request before the response and the saved settings could finish.

## The fix

The write and validation step now stops before the reload, and the reload became a separate step. A first attempt scheduled the reload right after the response, which still caused a brief 502 on the redirect that followed. The final version queues the reload as a delayed background job so it never overlaps the request that triggered it. A failed reload is logged and never turns a successful save into an error.

## Verifying

New tests cover the write, the separate reload and the delayed dispatch (2572/2572). On the development server we confirmed the redirect and next page load succeeded with the saved value, and that PHP workers restarted shortly afterward.
