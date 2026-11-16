# Phase 281 - DNS Zones With an Undo Button

DNS is the part of hosting where one wrong record takes a website or a whole mailbox offline, usually without a word of
warning. The DNS Zones page was a plain table, and the page for a single zone was a long list of editable fields with
nothing to say whether a change was a good idea or how to put it back.

## What changed

**A list that says what needs a look.** Every domain now shows whether its nameservers really point at this server, as
the rest of the internet sees them, and whether the records that keep its mail out of spam folders are in place.
Domains that need attention come first, and there is a search and a few filters for when there are many of them.

**A zone page that explains itself.** Four checks at the top (nameservers, website, mail, DNSSEC), then the records
grouped by what they are for instead of one long list. Anything that looks wrong carries a plain warning: a lifetime so
short that every lookup comes straight back to the server, a mail server given as an address when it must be a name, an
alias where one cannot work, two mail-policy records where there should be one. A missing mail record can be added in
one click with a recommended value.

**Every change kept, and a way back.** The zone's history shows who or what changed it and exactly which records, whether
that was an administrator, the account owner, or the panel itself renewing a certificate. Any earlier version can be
restored, after a page that shows precisely what would change. Records that something else is keeping up to date are
never rolled back, a restore is itself kept so it can be undone, and one that would leave a domain without its basics
says so first.
