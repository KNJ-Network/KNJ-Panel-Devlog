# Phase 51 - The SSL/TLS Section, Closed Out

Three items left on the SSL/TLS roadmap — CSR generation, a real certificate inventory, and
server-wide TLS policy configuration. All three shipped this session, and one of them got the kind
of live verification that actually matters: proving a policy change genuinely changes what the
server will and won't negotiate, not just that a setting got saved.

CSR generation on the admin side turned out to be the easy one — the account-side self-service page
already had a complete, correct implementation (a fresh private key and CSR generated entirely
in-process via PHP's openssl extension, no shell-out, key material never touching disk outside the
request). Reusing it for any hosted site from the admin side was a matter of wiring, not building.

Certificate inventory took the existing per-site SSL list and made it actually earn that name.
Before this, it showed status/issuer/expiry per site with no sense of urgency and no visibility into
the panel's own certificate at all — that lived on a completely separate page. Now every certificate
flags itself once it's inside 30 days of expiry, and the panel's own cert sits right alongside every
hosted site's, because "every certificate in use" should mean every certificate, not just the ones
serving customer traffic.

TLS policy configuration was the one worth being careful with — it changes the same `ssl_protocols`/
`ssl_ciphers` file every certificate on this server references, including the panel's own management
UI. Two fixed presets, Modern and Intermediate, matching Mozilla's own well-known nginx
configurations byte-for-byte — never free-text protocol or cipher input, so there's no way to
misconfigure this into something insecure even by accident. The provisioning script independently
re-validates the generated file's content against both known-good presets before writing anything,
the strongest version of this codebase's "never trust the caller" doctrine, since there are only two
valid states to check against.

The live verification was the actual test: applied Modern, then used `openssl s_client` against the
real server with `-tls1_3` and `-tls1_2` explicitly — TLS 1.3 negotiated cleanly, TLS 1.2 came back
with a real protocol-version alert, genuinely refused, not just cosmetically unavailable. Switched
back to Intermediate and confirmed both TLS 1.2 and 1.3 negotiated successfully again. That's the
difference between "the setting saved" and "the setting works" — this one actually works, proven
against the live server it was changing.
