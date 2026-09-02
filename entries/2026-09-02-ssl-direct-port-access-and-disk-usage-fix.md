# Phase 177 - Your Domain, Your Port, Your Certificate

Two small reports arrived in the same message: a customer's own domain on port 2083 threw a
certificate warning, and their dashboard's disk usage figure read "Unlimited" when it should have
read a real, small number. Neither was hard to find. One was a genuinely new feature; the other was
one word doing two jobs.

## One certificate, every domain

`https://<domain>:2083` (the account area) and `:2087` (the admin/controller area) both always served
whichever certificate happened to be configured for the panel's own hostname — fine for the panel's
own address bar, wrong the moment a customer typed their own domain with the port appended instead,
the classic direct-port access pattern this kind of panel is expected to support. The browser sees a
certificate for a name that doesn't match what it asked for, and shows exactly the warning that's
supposed to show up.

The fix doesn't need a new certificate at all. Every domain already gets its own certificate the
moment SSL is issued for it, and that certificate's first SAN entry is always the bare domain itself
— nginx just needed an SNI-based server block, one per domain per port, that answers on that domain's
name and serves that domain's already-existing certificate. `ensure_direct_port_vhost_blocks()` writes
exactly that, wired into every existing vhost-creation path plus the tail end of the SSL-issuance
action itself, so it activates the moment a domain's first real certificate lands — no separate step,
no waiting for the next deploy cycle.

Port 2087 deliberately got the same treatment as 2083, which is worth being explicit about: this
panel had an existing, deliberate rule that the admin entry point never sits on a customer-controlled
certificate, the same boundary this class of admin panel typically draws around its own login. Rather
than override that boundary quietly, the tradeoff was surfaced directly before writing a line of code,
and the answer that came back was clear — customers get their own domain and port on both surfaces,
not just the account one. Implemented exactly as asked, and the separate, pre-existing
`controller.<domain>` shortcut subdomain — a different surface entirely — was left completely alone.

## Zero doesn't always mean the same thing

The dashboard's disk and bandwidth figures share one small closure to turn a number of megabytes into
a human string. That closure had one rule: zero means Unlimited. Correct for a package's own
`disk_quota_mb` — a package with no configured cap really does mean no limit. Wrong for how much has
actually been *used*, where zero means exactly what it says: nothing yet, most obviously true of a
freshly restored account whose usage figures haven't been recalculated for the first time. One
formatter, two meanings for the same number, one of them silently wrong. Split into two formatters —
`$formatLimit`, unchanged, and a new `$formatUsage` that never treats zero as anything but zero — and
the two call sites that were using the wrong one now use the right one.

## Verifying both against something real

The certificate fix needed proof a browser screenshot couldn't easily give here — this environment's
own tooling wouldn't navigate to the non-standard port cleanly, so verification fell back to something
more definitive anyway: direct nginx vhost inspection over SSH and a real TLS handshake via
`openssl s_client`/`curl --resolve` against the live domain on both 2083 and 2087, confirming each one
now presents that domain's own certificate, not the panel's. SSH and the panel's own admin ports
stayed reachable throughout — checked, not assumed, given this touches how the admin area itself gets
reached. The disk-usage fix got the simpler check: a fresh account showing real zero-MB figures instead
of Unlimited, and a package with a genuinely unlimited quota still correctly showing Unlimited where
that's the true answer.

Tested.
