# Phase 64 - The Client Security Section, Closed Out

Two roadmap rows, one real gap, and a "primary domain only" limitation underneath both of them.

The roadmap read "SSL/TLS: certificates are live, CSR generation is next" and "SSL/TLS status
overview: single-domain view is live, bulk view is next." Reading the code first, both notes were
partly stale. CSR generation already existed, and the admin-side tool reuses it.

What was genuinely missing was that the account-side SSL page only looked at the account's
primary site, so an addon domain had no certificate visibility or control. Both roadmap rows were
the same limitation described two ways.

Fixed as one change. Every action now resolves its site from a posted identifier that is checked
against the account's own sites before anything runs. The page gained a bulk table across every
domain (status, issuer, expiry) with a "Manage" link per row.

It was verified with a disposable account that had two domains, rather than relying on
single-site test coverage. The table showed both, "Manage" switched the detail section, and CSR
generation ran for real against the addon domain.

Security section closed out. IP blocking, hotlink protection, directory login protection, and
account-level API tokens had already shipped in earlier phases.
