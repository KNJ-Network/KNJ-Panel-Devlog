# Phase 06 - The SSL Manager, and Two More Bugs Caught Live

M3 started as "SSL automation" — issue a Let's Encrypt certificate automatically when an
account's site goes live, no manual step. That part works, and it's been proven with a genuine
successful issuance against a real domain, not a simulated one.

But automation alone isn't what a real control panel gives an admin. A proper SSL/TLS Manager
needs a place to see certificate status across every account, and a way to install a certificate
bought elsewhere, not just the free automated kind. So M3 grew to match that:

- **Account side**: a status card showing the current certificate's issuer and expiry, a button
  to issue a Let's Encrypt certificate on demand, and a form to install a third-party
  certificate — paste the cert, the private key, and an optional CA bundle.
- **Controller side**: one page listing every site on the server, its certificate status,
  source, and expiry, with a per-site reissue button for admins.

Third-party certificate uploads are validated first — the certificate and key must match, the
certificate must not be expired, and it must cover the domain being installed to — before anything
privileged runs.

## Attacking the upload form before trusting it

Same rule as File Manager: nothing is done until it's been tested with hostile input. The upload
form was tested with injection-style input, a certificate for the wrong domain, and a certificate
paired with the wrong key. All were rejected cleanly with no privileged operation triggered, and a
valid certificate was then installed and confirmed served over a real TLS handshake.

That live testing surfaced two real bugs, both about how Let's Encrypt and a custom certificate
interact:

1. A failed reissue attempt on a site with a custom certificate could mark the site as "Issuance
   Failed" even though the certificate actually being served was untouched, so the status page
   showed the wrong state.
2. Removing an account cleaned up its Let's Encrypt certificate but not the files of a
   custom-uploaded one.

Both fixed, both re-verified live on a development server.
