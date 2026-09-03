# Phase 188 - The Shortcut That Called Itself

A domain reached on the panel port returned an error that the request was too large, even though the request had not grown.

## Cause

The feature that lets a domain use the panel port on its own certificate forwards requests internally to the shared panel. For a domain with its own matching configuration, the forwarded request could be routed back to itself, repeating until the request became too large.

## The fix

Forwarded requests now carry the panel's own name for routing, while the visitor's domain travels separately for cookies and generated links. The same repair applies to the webmail shortcut.

Fixing the loop exposed two smaller bugs that it had been hiding: a path-handling mistake that flattened deeper paths to the front page, and a port hint in a shape the framework does not read. Both are fixed.

## Verifying

We reproduced the original state on a disposable domain and confirmed the repair applied itself on the next routine update, and that requests now complete normally on the same domain and port.

Tested (2774/2774).
