# Phase 211 - The Answer Nobody Asked For, Rejecting Itself

Two bugs, found back to back, both in the same neighborhood: a form that hands someone credentials, and
the page that lets them use them.

## A password that failed to confirm itself

Auto-generating a password for an account was rejected with a "password confirmation doesn't match"
error. Nobody had typed a confirmation, since choosing Auto-generate hides the manual entry boxes. But
hidden fields are still part of the form. A browser that remembered a saved login could fill one of the
two and submit both, and the server's match check never asked whether either was relevant.

The fix was to make the fields honest about their relevance: they are disabled, not just hidden, so a
browser has nothing to autofill, and the confirmation check only runs when a manually typed password is
actually in play.

## The credentials that only worked half the time

A generated account comes with a username and a password, but only one of the two identifiers had ever
worked at login. Handed just the username, as happens when credentials are passed on to someone, the
login page only recognized an email address.

The fix widened what the page accepts rather than replacing it: it checks the input against either
identifier, and no longer requires the field to look like an email, since a browser's built-in
email-format check would reject a plain username before the server saw it. The rest of the login flow
doesn't depend on the shape of what is typed, so nothing else needed to change.
