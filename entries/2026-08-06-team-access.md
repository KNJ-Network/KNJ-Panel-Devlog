# Phase 66 - Team Access, and the 44-Controller Pass That Made It Real

The Preferences section's last two rows, "Sub-accounts" and "Team access", turned out to be one
feature described twice: the panel had no concept of more than one login per hosting account. It
was built as one feature.

Three decisions were made before any code. Invites go out by email (an existing panel user
accepts directly, and a new person creates their own login through the invite link, never sharing
the owner's password). Permissions are a custom checklist rather than a coarse full/limited
toggle. And once it was clear that making collaborator access actually work meant touching every
account-side controller, we chose to do the full pass in one change rather than land the invite
flow first and wire up enforcement later.

The permission catalog has nine keys, one per account navigation group, finer than a single
toggle and coarser than a per-action list. Managing other collaborators stays owner-only. The
work was a mechanical pass across the account-area controllers, running the full test suite
after each group. Owners noticed nothing change.

The navigation also hides groups a collaborator can't use, rather than showing dead links that
return an error.

Live verification hit a limit worth writing down. A disposable test account was created through
the live panel, and the invite itself went out and the collaborator record appeared correctly.
But on this development server the mailer was set to log-only, so the invite was never actually
delivered, and the accept-and-log-in half couldn't be proven through a browser this time. It
stayed covered by 16 automated tests (new-user accept, existing-user accept, wrong-user
rejection, double-accept rejection, and permission enforcement across several areas).

Disposable resources were cleaned up afterward; deleting the test account removed its mailbox
and invite with it.
