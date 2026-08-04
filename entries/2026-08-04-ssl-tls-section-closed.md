# Phase 51 - The SSL/TLS Section, Closed Out

Three items were left on the SSL/TLS roadmap — CSR generation, a real certificate inventory, and
server-wide TLS policy configuration. All three shipped, and one got the kind of live verification
that matters: proving a policy change changes what the server will and won't negotiate, not just
that a setting got saved.

CSR generation on the admin side was the easy one — the account-side self-service page already had a
complete implementation, generating the key and CSR in-process, so reusing it for any hosted site
from the admin side was wiring, not building.

Certificate inventory took the existing per-site SSL list and made it earn the name. Every
certificate now flags itself when it is inside 30 days of expiry, and the panel's own certificate
sits alongside every hosted site's, because "every certificate in use" should mean every
certificate.

TLS policy configuration was the one worth being careful with, since it affects every certificate
on the server, including the panel's own. It offers two fixed presets, Modern and Intermediate,
matching Mozilla's published recommendations, and never free-text protocol or cipher input, which
leaves less room to misconfigure into something insecure by accident. The privileged side
independently re-validates the result before writing anything.

The live verification was the actual test: applying Modern and using a TLS client against a
development server with specific protocol versions showed TLS 1.3 negotiating cleanly and TLS 1.2
refused. Switching back to Intermediate confirmed both negotiated again. That's the difference
between "the setting saved" and "the setting works."
