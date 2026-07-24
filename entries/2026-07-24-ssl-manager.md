# Phase 06 - The SSL Manager, and Two More Bugs Caught Live

M3 started as "SSL automation" — issue a Let's Encrypt certificate automatically when an
account's site goes live, no manual step. That part works, and it's been proven with a genuine
successful issuance against a real domain, not just a simulated one: `openssl s_client` showing
a certificate actually signed by Let's Encrypt.

But automation alone isn't what a real control panel gives an admin. cPanel and WHM both ship a
proper SSL/TLS Manager — a place to see certificate status across every account, and a way to
install a certificate bought elsewhere (Comodo, DigiCert, whoever), not just the free automated
kind. So M3 grew to match that:

- **Account side**: a status card showing the current certificate's issuer and expiry, a button
  to issue a Let's Encrypt certificate on demand, and a form to install a third-party
  certificate — paste the cert, the private key, and an optional CA bundle.
- **Controller side**: one page listing every site on the server, its certificate status,
  source, and expiry, with a per-site reissue button for admins.

Third-party certificate uploads get validated in plain PHP — cert and key actually match,
certificate isn't expired, and it actually covers the domain being installed to (checked against
both the certificate's CN and its Subject Alternative Names, wildcards included) — before
anything privileged ever runs. Installing it re-writes the site's Nginx configuration to serve
the uploaded certificate directly.

## Attacking the upload form before trusting it

Same rule as File Manager: nothing gets called done until it's been attacked on purpose. The
upload form got hit with shell-injection-style garbage in both fields, a real certificate for
the wrong domain, and a real certificate paired with the wrong private key. All three rejected
cleanly, with no privileged operation ever triggered — confirmed by checking the server directly,
not just trusting the HTTP response. A genuinely valid certificate was then installed for real
and confirmed being served over an actual TLS handshake.

That live testing surfaced two real bugs, both about what happens when Let's Encrypt and a
custom certificate interact:

1. Clicking "reissue" on a site that already had an active custom certificate installed — and
   having that Let's Encrypt attempt fail (as it will, for a test domain with no real DNS) — was
   flipping the site's status to "Issuance Failed" in the database, even though the real
   certificate on disk, the one Nginx was actually serving, hadn't been touched at all. The
   panel would have been lying to whoever looked at that status page.
2. Removing an account cleaned up its Let's Encrypt certificate but left a custom-uploaded
   certificate's files behind on disk — including the private key — with nothing to ever
   collect them.

Both fixed, both re-verified live after the fix.

## Next

M4 — App Installer, building one-click WordPress installs against real provisioned sites with
working SSL underneath them.
