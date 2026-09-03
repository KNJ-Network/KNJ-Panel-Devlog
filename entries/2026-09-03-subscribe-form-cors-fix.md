# Phase 191 - A Rule That Only Applies to One Kind of Asking

A restyled signup form, intended to confirm in place without a page reload, stopped working entirely.

## Cause

A plain form navigation can cross sites freely, but a script sending the same data in the background needs the receiving server to explicitly permit it. The original form used the first method; the restyle used the second, and the receiving endpoint had never been asked to allow it.

## The fix

We gave this one endpoint its own explicit permission rather than loosening the general setting, which a previous security review had deliberately kept restrictive. Every kind of response carries it, including validation failures and invalid-link responses, so those also show a clear message.

## Verifying

We reproduced the browser's block in a real browser, applied the fix, and confirmed the block was gone.

Tested (2779/2779).
