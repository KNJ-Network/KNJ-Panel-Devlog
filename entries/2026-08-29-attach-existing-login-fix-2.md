# Phase 157 - The Autofill Nobody Asked For

Fixed the "Attach to an existing login" uniqueness bug (Phase 156), shipped it, and got the user to
go re-test it live. Same form, same mode, brand new error: "The password field confirmation does not
match."

The password fields aren't even visible in that mode — the whole "Account owner login" password
block lives inside a `div` that gets `hidden` toggled by JS the moment "Attach to an existing login"
is selected. But `hidden` is `display: none`, not "removed from the form." The `password` and
`password_confirmation` inputs are still sitting in the DOM, still part of the same `<form>`, still
submitted on every POST regardless of what's visually shown.

And the browser doesn't care that a field is invisible. Chrome's saved-password autofill matched the
current origin, found a credential it had on file, and quietly dropped it into the hidden `password`
input — while `password_confirmation`, having no matching autofill heuristic of its own, stayed
empty. Two different values under one `'confirmed'` rule. Rejected.

This is the exact same bug shape as Phase 156, just one field group over: validation rules that were
written assuming "if the user didn't touch it, it'll be null" don't hold once something *other than
the user* can populate a field. `required_if:owner_mode,new` protects against the field being
enforced-empty; it does nothing about a rule like `confirmed` still evaluating against whatever
happens to be sitting there.

Fix mirrors Phase 156 exactly: stop validating `password_mode`/`password` at all outside
`owner_mode === 'new'`, rather than trying to make the existing rules tolerant of garbage. A field
that's genuinely irrelevant to a code path shouldn't have any rule watching it in that path, no
matter how "safe" that rule looks in isolation.

Added two things beyond the validation fix, since the browser behavior itself isn't going away:

- `autocomplete="new-password"` on both fields — the standard signal to stop a password manager from
  treating this as a login form worth autofilling.
- The existing-login toggle now explicitly clears both fields' values in JS when switching modes, not
  just their `required` flags.

Neither of those is the real fix — a resubmitted form, a different browser, a different heuristic
could all still land a stray value in there. The validation-layer fix is what actually makes it safe;
the other two just make the common case quieter.

New regression test posts `owner_mode: existing` with a `password` value and a missing/mismatched
`password_confirmation` — exactly what a browser autofill leaves behind — and asserts the account
still creates cleanly.

Tested (2508/2508).
