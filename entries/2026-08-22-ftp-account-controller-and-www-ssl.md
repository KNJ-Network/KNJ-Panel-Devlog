# Phase 119 - ftp/account/controller shortcuts, and closing the www SSL gap

A follow-up to the previous day's webmail DNS-shortcut work, picked up from the DNS-only comparison
audit: `ftp.<domain>` and account and controller panel shortcuts were the next candidates, and a
real SSL bug was found and closed alongside them.

## ftp.<domain>

The smallest version of this pattern: FTP is built and verified, so this is only a DNS record with
the same toggle as `mail` and `webmail`. No web server work is needed since FTP is not HTTP.

## account.<domain> and controller.<domain>

Shortcuts to the account panel and the admin panel, named for this project's own areas. The
mechanism matches the webmail fix: two new toggleable DNS records, redirect locations in the shared
vhost snippet, and catch-all vhosts for the shortcut names.

## The www SSL gap, a real bug found while researching

Every account's vhost has always served `www.<domain>`, but SSL issuance only requested the apex
domain, so `www` was served with a certificate that did not cover it. Issuance now requests both
names in one call. Because multi-name issuance is all-or-nothing, a fallback retries apex-only if
`www` does not resolve to this server yet, so an account never ends up with no certificate.

## Deliberately not done this pass: bundling mail/webmail/account into the same certificate

Doing it safely means those hostnames need to be real entries on the customer's vhost, which touches
every vhost-creation path in the provisioning script. That is separate surface with real risk to
live HTTPS, not something to bundle into the same pass. The admin-panel shortcut should stay out of
any customer certificate in any case, since it is a server-wide surface. The `www` fix was safe to
do immediately because `www` was already in every vhost.

## Verified

Full local suite (1,710 tests) and the code-style check green, plus a syntax check on the
provisioning script, which has no automated test harness; that part was verified live instead.

## Live-verify addendum (same day)

Shipped as v0.16.48 and checked against the dev server. The new DNS records appeared after
restoring default records and resolved correctly. The redirects required re-applying settings once
to pick up new code, since the shared snippet is only rewritten when an admin re-saves those
settings; real installs get the shortcut once one of those pages is saved after upgrading.

The `www` fix itself had a bug: re-issuing for a domain with an existing certificate still produced
an apex-only result, because the certificate tool refuses to add a name to an existing certificate
without an explicit expand option in non-interactive mode, and the fallback then masked the failure.
Initial hand-tracing had no real certificate history to test against. Adding the option fixed it,
shipped as v0.16.49, and the test domain's certificate now covers both names with no mismatch.
