# Phase 287 - A Lookup That Answers Questions

The DNS Lookup page was a form. You typed a name, picked a record type, and got a block of raw output back, which meant you
had to know what to look for and how to read it. Earlier today it learned to ask several places at once, but it still
opened empty and still left the reading to you.

## What changed

It now opens with something to do. The domains hosted on the server are listed with the recently changed ones first, next to
what you are watching and what you looked up lately. The search box takes anything you paste: a plain name, a web address
with its path and port, or an address, which turns into a reverse lookup.

Three questions have their own answers. For "is this domain set up right?", one check runs the whole list: who answers for
the domain and whether they all agree, whether the website and mail records are in place, whether sending is trusted, and
whether the security records are consistent. Each row says in words what it found, and for a domain hosted here it says
whether the internet shows the same as the server's own copy. For "has my change spread?", the page shows how many of ten
places already have the new answer, what happened in order, and when the old answer will be gone at the latest. And for
"tell me when it is done", a change can be watched in the background: an email arrives once when every resolver has it, or
once if it still has not after a day.

Customers get the same page for their own domains. Their lookups are only ever compared with their own records, and a
watch belongs to the person who asked for it.

Reading it over, one thing was wrong: a name that began with a dash could be handed to the lookup tool as an option instead
of a name. The customer page already refused that; the administrator page did not. Both now share one check.
