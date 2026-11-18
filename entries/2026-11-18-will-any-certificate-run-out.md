# Phase 300 - Will Any Certificate Run Out Unnoticed

The SSL/TLS page listed every site with a status, but a failed certificate showed only a red label and nothing about why,
nothing said whether automatic renewal was actually running, and the server's own certificate lived on a separate page. Worse,
the expiry dates were blank for every certificate that was issued automatically, because nothing had ever recorded them. The
page showed dashes, the dashboard could not see an expiry coming, and the email that warns an account owner their certificate
is about to expire could never be sent.

## What changed

The page is now built around one question: will any certificate run out without anyone noticing? A list at the top puts the ones
that need attention first, each with the reason in plain words and a button: a name whose DNS does not point at the server yet,
a connection that timed out, a rate limit, a certificate that should have renewed and has not. The server's own certificate sits
beside it, with a check that the renewal timer is running and how its last run went.

The expiry date, who issued each certificate and which names it protects are now read from the certificates themselves, so they
stay true after a renewal and the warnings that depend on them finally work. An administrator is alerted once when a certificate
is two weeks from expiring and has not renewed.

The protocol policy page now says who each choice would lock out, and shows what the server really accepts right now rather than
only what was saved.
