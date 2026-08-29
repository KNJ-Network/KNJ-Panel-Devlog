# Phase 156 - The Uniqueness Check That Didn't Understand the Point

A live bug report: Create Account, with "Attach to an existing login" selected instead of "Create a new
login," failed every time with "The owner email has already been taken," for any domain and any existing
user.

The message was not wrong, exactly; it just did not apply. The owner-email field's rule was "required
only in new-login mode, nullable, valid email, unique among users." "Required only in new mode"
stops the field being required outside that mode, but does nothing to stop the uniqueness check
running against whatever value is in the field, and attaching to an existing login means reusing an
email that already belongs to a user. The rule was rejecting exactly what that mode exists to produce.

We confirmed the fix has no side effect: when an existing user is passed, the provisioning service
resolves the owner from that user and never reads the owner name and email parameters, so gating the
rule behind new-login mode cannot overwrite anyone's real email.

The more interesting finding was in the test suite: nothing had ever exercised existing-login mode end
to end. Every account-creation test posted new-login mode, so the feature shipped and broke inside a
blind spot, not through a later regression. We added a test that creates an existing user, attaches a
brand-new domain to them, and asserts the account lands with the right owner and no third user is
created.

Tested (2506/2506).
