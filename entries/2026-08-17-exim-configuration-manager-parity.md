# Phase 96 - Two Small Ones: User Manager and the Postfix Config Gap

After audit #11, two parity items remained: User Manager and Exim Configuration Manager. Neither
needed what its name implies.

## User Manager was already done under a different name

cPanel's User Manager lets someone else help manage an account without handing over the whole login.
That's what Manage Team already does, with scoped collaborator accounts, built for how this panel
authenticates. It was marked closed rather than building a second mechanism for the same problem.

## Exim Configuration Manager, translated to what this install runs

The panel runs Postfix, not Exim, by deliberate choice, so the honest version of this item is
whether the Postfix-equivalent control surface exists. The Mail Settings page already covered the
basic anti-abuse controls but stopped short of the policy-level ones. Four were added: HELO/EHLO
hostname validation, sender-domain verification, a custom SMTP greeting banner, and incoming TLS
policy (opportunistic or required). All four apply through the same validated
backup-and-revert-on-failure path as every other write on this page.

The free-text banner field needed input validation. The value is checked where it's entered and
again independently in the provisioning script, rather than trusting the first check alone.

This is deliberately not a raw Postfix editor. Postfix has several hundred tunable parameters, and
this panel exposes a curated surface for every service.

## The check the test suite couldn't have caught

Live-testing against the development server's real Postfix found that an ordinary banner containing
spaces was rejected. The pattern used to validate printable characters didn't mean what it looked
like in a shell, and the automated tests couldn't catch it because they mock the command layer. It
was replaced with the standard POSIX printable class, verified against ordinary and invalid values,
then re-verified live: all four new settings applied and read back correctly, then were reset to
defaults.

## What's left

The full suite is green (1419 tests, including new ones for the four controls and the validation
case), code style is clean, and the change is deployed and verified.
