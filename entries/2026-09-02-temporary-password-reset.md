# Phase 170 - The Password Nobody Should Be Able to Reuse

The Reset Password page could generate or set a password, shown once on screen, and never emailed it. We added an option to email the new password, and a temporary-password mode: one-time use, time-limited, and forcing a change at first login. Because this is a security feature, we planned it carefully first.

## Design

- The existing "Forgot password?" flow is separate and left untouched.
- The tracking fields are never mass-assignable and are reset by every reset mode, so a normal reset clears any pending temporary state.
- Expiry and prior-use checks happen only after the password itself has been verified, so a wrong guess never reveals whether a temporary password exists.
- Marking it used applies only to real logins, so an admin acting on behalf of an account cannot consume it before the owner does; a dedicated test covers this.
- A forced-change page follows the pattern of existing gate checks, and does not require the old password, since using the temporary one already proved possession.
- Temporary mode always emails and never shows the password on screen.

## Verifying

2610/2610 passing. On the development server we ran the full flow end to end: delivery, forced change, rejection of a weak replacement, rejection of reuse, and normal access afterward. We also fixed Manage Servers showing "Never" for the main server's own Last Seen.

Tested (2610/2610).
