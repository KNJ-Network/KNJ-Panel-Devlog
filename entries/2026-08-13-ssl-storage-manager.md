# Phase 76 - A Certificate Repository Instead of a Site List

The next item on the gap list was an SSL Storage Manager. The existing SSL page listed every site's
current certificate, but that is also its limit: it only knows about a certificate still attached
to a site that still exists. A certificate left behind after its domain was removed, or a replaced
certificate whose old files weren't cleaned up, was invisible to the panel. This is the other half:
a repository of every certificate the server actually holds, independent of whether anything uses
it.

## Reading the filesystem instead of the database

A new read-only provisioning action reports the certificates actually present, with subject,
issuer, expiry, and fingerprint. The panel then cross-references the results against its own site
records, so the provisioning side only ever reports what is really there.

Every row is in one of three states: attached to a site, the panel's own protected service
certificate, or orphaned. Only orphaned entries get a delete button, and the delete request is
re-checked against the live cross-reference before anything is removed, never trusted from what
the page showed when the click landed.

## A bug only a real certificate would show

This was tested thoroughly against faked output and then broke against the one real certificate on
the server: the real tool's output label differed in letter case from every reference example, and
one missing field took the whole action down. It now matches case-insensitively and lets each field
fail independently. The lesson: the only way to know a tool's output format for certain is to run
it against the real thing.
