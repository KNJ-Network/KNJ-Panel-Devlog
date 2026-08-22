# Phase 121 - Mail client autoconfiguration

The second item off the "genuinely zero" list. Comparable panels let a mailbox owner point
Thunderbird, Outlook or a phone mail app at their account with just an address and password, with
no manual typing of server names and ports. This closes the same gap, scoped to what this mail
stack actually runs.

## Two halves, one toggle

**RFC 6186 SRV records** for IMAPS and submission are added to every new zone's defaults, with the
same toggle pattern as the rest of Zone Templates (default on). The target hostname comes from the
same lookup KNJ Webmail uses, so it respects Mail Only satellite linking: if a Mail Only server is
active, the records point there; otherwise they point at this box.

**Mozilla autoconfig XML** is served at the standard well-known path (plus the legacy path some
clients still check) directly on the customer's own domain, with no separate subdomain needed. It
is generated at account-provisioning time through the same per-account mechanism the error-page and
MIME-type features use, with no changes to vhost creation.

## Only publishing what actually exists

Only the client-facing services that are actually available are published: IMAPS and STARTTLS
submission. POP3 and SMTPS are not advertised, since publishing discovery for a service nothing
answers on would hand a client a broken fallback.

## Deliberately out of scope this pass

The classic `autoconfig.<domain>` subdomain and Microsoft's Autodiscover schema were skipped. They
need genuinely new infrastructure or a larger compatibility surface than this pass warranted.

## Verified

4 new tests plus updated assertions; full suite 1,735 and the code-style check green.

## Live-verify addendum (same day): a real, pre-existing DNS bug

Shipped as v0.16.52 and checked against a test domain with real DNS queries and a real HTTPS
request. The autoconfig XML was correct on the first try. The SRV records were not: the target came
back with the customer's zone name appended. The zone service made CNAME, MX and NS targets fully
qualified before writing them, but SRV targets were never included, so DNS treated them as relative
to the zone. This pre-dated the feature: any SRV record added manually with an external target had
the same problem, and it only surfaced when the panel generated SRV records pointing at a different
domain. A helper now fully qualifies just the target field of SRV values, with a regression test.
Shipped as v0.16.53 and re-verified live.
