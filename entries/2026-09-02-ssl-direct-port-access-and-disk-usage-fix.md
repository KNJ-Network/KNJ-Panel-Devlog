# Phase 177 - Your Domain, Your Port, Your Certificate

Two reports arrived together: a customer's own domain on the account port showed a certificate warning, and the dashboard's disk usage read "Unlimited" instead of a real number.

## One certificate, every domain

Visiting a domain with the panel port appended presented the panel's own certificate. Each domain already has its own certificate, so the web server now answers per domain with that domain's certificate. It is set up wherever a domain's site is created and when its certificate is issued. We decided customers should get this on both the account and admin surfaces, and left the separate `controller.<domain>` shortcut unchanged.

## Zero doesn't always mean the same thing

A single formatter treated zero as "Unlimited", which is right for a package limit but wrong for usage. It is now two formatters, and a fresh account shows zero used.

## Verifying

We inspected the configuration and performed real TLS handshakes against a live domain, confirming each port presents the domain's own certificate. Administrative access stayed available throughout. The disk-usage fix was checked on fresh and unlimited packages.

Tested.
