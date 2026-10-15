# Phase 263 - The Tool Nobody Should Be Able to Find

The last open finding from the penetration test was the database administration tool. It still needed a valid
login, but nothing about it needed to be visible to a stranger at all.

## What it does now

The tool now answers anyone without a signed-in database session exactly as it would any address that does
not exist. Customers still open it with one click from their account panel and never see a login form.

Testing it as a customer found one more thing: the tool's own session handling quietly undid some of the
cookie protections we had set. One configuration line now keeps them, and two warnings people had been seeing
on its front page are gone for the same reason.

## The upgrade that broke it

Upgrading our own production server turned that tool into an error page. The cause was a flaw in how
upgrades were applied: one run could use the previous release's steps, so a second run was needed to apply the
new ones. A customer cannot do that, since after one click the panel says it is up to date.

The fix is in two parts. The upgrade now hands over to the new copy of itself, so one run applies everything.
And the web server rules are never written before the file they depend on exists. Our own server was repaired
by taking the next release, and a new install gets the current installer and the latest release anyway.
