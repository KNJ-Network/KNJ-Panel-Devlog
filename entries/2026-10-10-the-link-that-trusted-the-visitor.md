# Phase 252 - The Link That Trusted the Visitor

The first of four security audits in one day started with a question about something ordinary: where does
the link in a password-reset email actually come from?

## Built from whoever asked

The panel answers to customers' own domains, so the address a link is built from deserves care. Links the
panel builds now use a short list of hostnames it knows are its own, rather than information supplied by
the request.

## The same afternoon, a few more of the same shape

The pattern the first audit kept finding: a value that is fine in the normal case, trusted a step too far.
Several inputs were accepted a little too readily. Each got a check at the point the value is accepted and
a second check in the privileged step behind it, with a test that fails on the old code.
