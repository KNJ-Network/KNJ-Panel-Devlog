# Phase 65 - The Client Advanced Section, Closed Out, and a Second Stale Roadmap Row

A one-item section, plus a roadmap claim that turned out to be wrong.

The DNS Lookup tool needed almost no new design: the Controller area already had a lookup tool, so
it was reused on the account side. It only queries public DNS, so there is no account-specific
boundary to enforce.

While checking what was left in Preferences, the "Password & two-factor" row said self-service
password change didn't have its own page yet. It does, on the My Profile page next to the
two-factor setup. Two-factor itself (authenticator app, recovery codes, and an emailed one-time
code) was already correctly marked. Only the password half of the note was out of date.

Advanced section closed out. Preferences stays open: Sub-accounts and Team access were a real
data-model gap rather than a stale row, since the panel had no concept of more than one login per
hosting account, and that was scoped as its own build.
