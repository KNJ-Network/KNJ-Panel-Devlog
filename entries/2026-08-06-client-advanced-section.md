# Phase 65 - The Client Advanced Section, Closed Out, and a Second Stale Roadmap Row

A one-item section, plus a roadmap claim that turned out to be simply wrong.

The DNS Lookup tool itself was almost zero new design work — the Controller area has had a `dig`-backed
lookup tool since earlier this session (Track DNS: hostname + record-type form, shell out to `dig
+noall +answer +additional`, same hostname-validation regex), and there was no reason to invent a
second approach for the account side. Copied the controller and view as-is into the `Account`
namespace, no ownership check needed — it only ever queries public DNS infrastructure, nothing
account-specific, so there's no boundary to enforce that the Controller-side version didn't already
need either.

While researching what was actually left in Preferences, the same session pattern showed up again:
the "Password & two-factor" row said self-service password change didn't have its own page yet. It
does — it's been sitting on the My Profile page, right next to the two-factor setup, since before
this build session even started. Two-factor itself (authenticator app, recovery codes, an emailed
one-time code as a third path Fortify doesn't provide out of the box) was already correctly marked.
Only the password-change half of the note was wrong, and only because nobody had gone back to update
it after it shipped.

Advanced section closed out — five for five. Preferences stays open: Sub-accounts and Team access
are still genuinely missing, and unlike everything closed out so far this phase, that one is a real
data-model gap, not a UI gap or a stale roadmap row — this codebase has no concept of more than one
login per hosting account today. That's a bigger, more careful build, and it's being scoped properly
before any code gets written rather than bolted onto this phase.
