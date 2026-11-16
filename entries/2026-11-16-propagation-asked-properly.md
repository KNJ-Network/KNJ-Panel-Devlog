# Phase 284 - Propagation, Asked Properly

The DNS menu had a Propagation Checker. You gave it a domain and an address, and it told you whether the domain's A record
had "propagated". What it actually did was ask the server's own resolver, once. That is a smaller version of the DNS
Lookup page next to it, and it was not really answering the question: propagation means some resolvers around the
internet still hold the old answer while others have the new one, and you cannot see that by asking one of them.

## What changed

The checker is gone, and DNS Lookup now does the job properly. It asks the domain's own nameservers what is published,
which is the truth, and asks Cloudflare, Google, Quad9 and the server's own resolver what they currently hold, all at
once, for any kind of record. Give it the value you expect and it answers in plain words: it has propagated; it is
published correctly and still spreading, so wait; or it is not what you expected, and waiting will not help, so fix the
record. That last distinction is the useful one, because it tells you whether to wait or to act.

A resolver that did not answer at all is shown as that, rather than being counted as holding an old answer. The full
answer from the server is still there for anyone who wants the raw output. The places that used to link to the checker
now open DNS Lookup with the domain and the server's address already filled in, and the old address redirects.
