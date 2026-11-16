# Phase 286 - Mail That Gets Believed

The Email Deliverability page had a table that said "Valid" next to each domain's SPF, DKIM and DMARC. That meant the
record existed and was shaped correctly, which is not the same as mail being believed. A domain could show Valid for DKIM
while nothing was signing with that key, or while the published key was an old one. It also looked up every domain's DNS
live each time it opened, so it got slower with every domain hosted, and when it found a problem it told you to go and
fix it somewhere else.

## What changed

The page now checks the things a receiving mail server checks. SPF is worked out the way a receiver works it out, for the
address mail really leaves from, so a record that exists but does not allow the server is called out. DKIM compares the
key in DNS with the key the mail server signs with, so a published record that nothing uses, or a key that no longer
matches, is caught. DMARC says whether it enforces anything or only watches.

The sending server is checked once, in its own card, instead of being repeated for every domain: where mail leaves from,
whether its reverse DNS comes back, whether it signs, and whether its address is on a blocklist.

Results are kept and refreshed in the background, so the page opens at once however many domains there are. A What to fix
tab lists every problem across every domain in plain words, worst first, and All domains is a table you can filter and
search. Each domain has its own page that explains each result and shows the record that would fix it.

And there is a button for it. Fixing SPF adds the server to the record and keeps the rest, and refuses when the domain has
two records or the record points somewhere else, because guessing there would break mail. Fixing DKIM publishes the right
key or makes one. Fixing DMARC starts with watching and only moves to quarantine once SPF and DKIM both pass, and it never
goes stricter than that by itself. Every fix goes through the normal zone change, so it appears in the zone's history and
can be undone, and the buttons are switched off, with the reason, for a domain whose DNS lives somewhere else.

## One more check: the name it gives

Reverse DNS showed "Confirmed" whenever the address and its reverse name agreed with each other. Receiving mail servers
also compare that name with the one a server announces when it connects to them, and a mismatch costs trust even when the
reverse DNS itself is fine. The sending-server card now has a row for it. It reads the name the server announces and says
whether it points at the sending address and matches the reverse DNS. When it does not, the page says which of the two to
change and who changes it, rather than offering a button, because one is a whole-server setting and the other belongs to
the provider that owns the address. The health check on the Diagnostics page has the same row.
