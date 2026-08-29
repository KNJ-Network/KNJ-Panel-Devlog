# Phase 157 - The Autofill Nobody Asked For

After fixing the "Attach to an existing login" uniqueness bug (Phase 156) and shipping it, a live
re-test of the same form and mode gave a brand-new error: "The password field confirmation does not
match."

The password fields are not visible in that mode: the "Account owner login" password block is hidden
by JS when "Attach to an existing login" is selected. But hidden is `display: none`, not removed from
the form, so the fields are still submitted on every POST.

The browser's saved-password autofill matched the origin and dropped a credential into the hidden
password input, while the confirmation field, with no autofill heuristic of its own, stayed empty. Two
different values under one "must match" rule.

This is the same bug shape as Phase 156, one field group over: validation rules written assuming an
untouched field is null do not hold once something other than a person can populate it. "Required only
in new-login mode" stops the field being required elsewhere, but not a "must match" rule evaluating
whatever happens to be there.

The fix mirrors Phase 156: stop validating the password fields at all outside new-login mode. A field
irrelevant to a code path should have no rule watching it in that path.

Two further changes make the common case quieter: `autocomplete="new-password"` on both fields, and
clearing both fields' values in JS when switching modes. Neither is the real fix; the validation change
is what makes it safe.

A new regression test posts existing-login mode with a password and a missing confirmation, exactly
what an autofill leaves behind, and asserts the account still creates cleanly.

Tested (2508/2508).
