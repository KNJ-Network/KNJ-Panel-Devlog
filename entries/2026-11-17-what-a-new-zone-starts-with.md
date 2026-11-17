# Phase 289 - What A New Zone Starts With

Zone Templates was a list of ten boxes. Ticking one decided whether every new domain got a particular DNS record, but
nothing said what the record was for, what stopped working without it, or what a new domain would actually end up with, and
the records that are always added, because a domain cannot work without them, were not mentioned at all. It also only ever
touched new domains. The one tool for bringing older domains into line handled three mail records and nothing else, so a
record you wanted on every domain had no way onto the ones you already had.

## What changed

The switches are grouped by what they are for, each with a plain sentence, and turning off one that matters says what will
stop working. Beside them is a preview of exactly what a new zone for a sample domain will contain, built by the same code
that creates a real zone, so the two cannot drift apart, and it updates as you click.

Every switch also shows how many of your existing domains already have it. Where some do not, one click opens a review of
every domain that would change and what it would get. Nothing happens until it is confirmed, a record a domain already has
is never touched, one domain that cannot be written does not stop the others, and each change is kept in that domain's
history so it can be undone. The same works for your own extra records, which could not be added to older domains at all
before, and adding one now has clickable placeholders and a live line showing what a new domain would get.

The separate DNS Settings page, a handful of timing values that belonged with this one, is now a tab here, with each value
shown in plain words next to the number.
