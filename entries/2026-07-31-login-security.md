# Phase 38 - Locking the Front Door: Brute-Force Protection and 2FA

Second of the four v1.0 items: protecting the panel's own login. Intrusion prevention already
protected SSH on every server, but the panel's login form relied only on the app's built-in
throttle.

**Brute-force lockout reuses infrastructure already proven for SSH**, rather than inventing a
parallel app-level system. Repeated failed panel logins are recorded and picked up by the same
intrusion-prevention tooling, which bans the offending IP at the network level. It installs itself
on every server through the same mechanism that keeps the panel's other components current.
Live-testing found that the first configuration silently never saw the app's log entries; it was
corrected and confirmed by watching a real failed login being counted.

**Two-Factor Authentication turned out to be mostly already there.** The auth framework supported
it, but the feature had never been wired up. It's now opt-in for every role (admin, reseller,
account owner), on a new shared "My Profile" page reachable from both the Controller and Account
sidebars: QR code setup, a confirmation step before it turns on, recovery codes, and disable.

**A third fallback was added**: an emailed one-time code, for someone who's lost their authenticator
and never saved their recovery codes. All three paths behave identically on success.

Every path was tested against real generated codes: enable, confirm, disable, a login-time
challenge, a recovery code that works once and is refused the second time, and an emailed code
round-tripped through the mail log. That caught two bugs: replay protection rejecting a second code
from the same time window, and an invalid nested form that made the "email me a code" button submit
the wrong endpoint. Both were fixed and confirmed in a real browser before shipping.
