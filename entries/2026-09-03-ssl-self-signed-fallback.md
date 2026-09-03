# Phase 187 - A Wrong Answer Beats a Random One

A recent incident showed that when a domain's certificate has not been issued yet, an HTTPS visit fell through to the server's default site and could show unrelated content under that domain's name.

## The fix

Certificate issuance will sometimes fail for ordinary reasons such as DNS propagation delay, so the aim is a safer failure. When a real certificate cannot be issued, the panel now generates a temporary self-signed one for that domain. Visitors see a certificate warning attached to the correct site instead of another site's content.

## Automatic and tidy

A short background cycle retries issuance and swaps in the real certificate as soon as it is available. A failing domain keeps its existing temporary certificate, and the temporary configuration is removed cleanly when the real one lands.

## Verifying

We tested with a real account and a domain that cannot resolve, running the real steps through to an actual connection. The right site appeared with its own name on the warning, and a retry changed nothing that did not need to change.

Tested (2758/2758).
