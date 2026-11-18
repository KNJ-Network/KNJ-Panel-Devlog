# Phase 301 - Who Can Call The API, And What Have They Done

The API Tokens page was one form and a table of your own tokens. It could not say who else on the server had a token, a token
never expired, and nothing recorded what a token had actually been used for. If a billing system's token leaked, there was no
way to limit where it worked and no way to see it being misused.

## What changed

The page now starts from what a token is for. A billing system, a monitor and a provisioning script each get a sensible starting
set of abilities, the ones that can create or remove things are marked, and a token can be given an expiry and a list of
addresses it may be used from. Anything else is refused, even with the right token.

Administrators can see every token on the server, whoever made it, with what it can do and when and from where it was last used,
and revoke any of them. A new Activity tab lists every call made to the API: which token, what it asked for, how it ended and
where it came from, so a token trying something it was never given stands out. Nothing that was sent or received is kept.

A token that is about to expire, or has not been used for months, is called out on the page, and administrators are emailed a
week before a token expires, because whatever relies on it stops without warning.
