# Phase 297 - One Page For Who Gets In

The Firewall page was a long list of rules with the system's own mixed in among yours and nothing saying what any port was
for, a block of raw text from the brute-force protection, a box for each protection, and the list of addresses that must never be
blocked. A second page, Access Control, did a closely related job: choosing which addresses may reach SSH, the panel, FTP, mail
and DNS. Two pages for the same question, neither answering "why is this address blocked, and when will it be let back?".

## What changed

They are now one page with five tabs. The ports the server leaves open appear in a single table grouped by what they are for, each
named plainly, with your own rules kept apart. Every blocked address appears in one table with the reason, when it was blocked and
when the block lifts, and a button to lift it or to make sure it can never happen to that address again.

Access Control became a tab. Its one real danger, allowing a single address for SSH or the panel and so closing the door on
yourself, is now caught: if the first address you add is not the one you are using, the page says so and will not save until you
confirm you mean it. The page also shows the address you are connecting from throughout, and whether it is on the never-block
list, because that is the number that matters before changing anything here.
