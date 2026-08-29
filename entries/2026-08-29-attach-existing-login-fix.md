# Phase 156 - The Uniqueness Check That Didn't Understand the Point

Got a live bug report on the production stack: Create Account, with "Attach to an existing login"
selected instead of "Create a new login," fails every single time with "The owner email has already
been taken." Any domain, any existing user, same result.

The message wasn't wrong, exactly. It just didn't apply. `owner_email`'s validation rule was:

    'owner_email' => ['required_if:owner_mode,new', 'nullable', 'email', 'max:255', Rule::unique('users', 'email')],

`required_if` correctly stops the field from being *required* outside "new" mode. It does nothing to
stop `unique` from still running against whatever value happens to be sitting in that field — and
attaching to an existing login means reusing an email that's already in `users`, by definition. The
rule was checking for exactly the condition that mode exists to produce, and rejecting it.

Confirmed the fix carries no side effect: in `AccountProvisioningService::createAccount()`, when an
`existingUserId` is passed, the method resolves the owner via `User::findOrFail($existingUserId)` —
`ownerName`/`ownerEmail` are never read in that branch at all. They're dead parameters once an
existing user is attached, so gating the `unique` rule behind `owner_mode === 'new'` doesn't open any
path to silently overwriting someone's real email.

The more interesting finding was in the test suite: nothing had ever exercised `owner_mode=existing`
end to end. Every existing `AccountCreationTest` case posted `owner_mode: 'new'`. The "attach to
existing login" feature shipped, and then broke, entirely within a blind spot — not a regression
introduced later, a path that was never actually driven by a test in the first place. Added one:
create an existing user, attach a brand-new domain to them, assert the account lands with the right
`user_id` and no third user gets created.

Tested (2506/2506).
