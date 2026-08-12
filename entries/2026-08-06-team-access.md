# Phase 66 - Team Access, and the 44-Controller Pass That Made It Real

The Preferences section's last two rows — "Sub-accounts" and "Team access" — turned out to be one
feature described twice. "Create additional logins sharing email/FTP/file access" and "Invite other
people to access this account with scoped roles" are the same underlying gap: this codebase had no
concept of more than one login per hosting account. Built as one feature rather than two.

Three decisions got made with the user before any code: invites go out by email (an existing panel
user accepts directly, a new person creates their own login through the invite link — never sharing
the owner's own password); permissions are a fully custom checklist, not a coarse full/limited
toggle, matching the `reseller_permissions` pattern already used elsewhere in this codebase; and —
the big one — once it became clear that making collaborator access *actually work* meant
touching every account-side controller, the user was asked directly whether to do the full pass now
or land the invite/accept flow first and wire up enforcement later. Full pass now, in one commit.

The permission catalog is nine keys, one per account nav group (files, domains, email, databases,
metrics, security, software, advanced, preferences) — coarser than a per-action ACL, finer than a
single toggle, and deliberately excludes anything that could let a collaborator manage other
collaborators. Team Access management itself stays hard owner-only: its controller resolves the
account through its own separate `$request->user()->accounts()->firstOrFail()`, never through the
new collaborator-aware resolver, so no permission combination can ever reach it.

That resolver is the actual architecture change. Every one of the ~44 `Account\*Controller` files had
its own independent `$request->user()->accounts()->...->firstOrFail()` call — the only thing that
made a controller reachable by the account's owner and nobody else. Rather than teach each one
individually about collaborators, two methods went on the shared `AccountAreaController` base every
one of them already extends: `accountQuery()` (returns `Account::accessibleBy($user)` — the owner's
own account, or any account they're an accepted collaborator on) and `requirePermission($account,
$key)` (a no-op for the owner, a 403 for a collaborator missing that key). Then a mechanical pass:
swap the call site, add one permission check, controller by controller, group by group, running the
full suite after each group rather than only at the end. Owners never noticed anything changed —
the no-op path is exactly their old behavior.

The nav builder got the same treatment for a different reason. A collaborator without `databases`
shouldn't see a dead "MySQL Databases" link that just 403s when clicked — but reusing the existing
"Soon" badge for that would have been wrong, since that badge already means something else (a
package feature not yet included). A permission-gated group is simply absent from the nav instead,
via a `$group()` closure that returns `null` and gets filtered out.

Live verification hit a real wall partway through, worth writing down honestly. A disposable test
account and a real mailbox on it were created through the live Controller and account panel exactly
as a real customer would, and the invite itself went out for real — "Invitation sent" from the
actual deployed UI, a real `account_collaborators` row, the collaborator showing up correctly in
"People with access." But reading that invite back through either webmail client
(`panel.dev.knj.network/roundcube/` and the panel's own KNJ Webmail) failed with "Connection to IMAP
server failed," even though Service Status reported Dovecot as Active and a restart through the
panel's own Service Manager didn't fix it. Checking Mail Queue and Mail Delivery Reports next showed
the real cause was upstream of that: nothing had ever reached Postfix for that recipient at all —
`.env.example` still defaults `MAIL_MAILER=log` on this dev box, so the notification was written to
the app's own log rather than actually mailed. Expected dev-environment behavior, not a bug in this
phase's code, but it meant the accept-and-log-in half of the flow couldn't be proven through a real
browser session this time. That half stayed covered by the 16 tests written for it — brand-new-user
accept, existing-user accept, wrong-user rejection, double-accept rejection, and permission
enforcement spot-checked across three different keys and controllers — run against the identical
code that's now live. The broken IMAP connectivity itself is a separate, real finding, flagged for
its own follow-up rather than folded into this phase.

Disposable resources cleaned up afterward — deleting the test account cascades to its mailbox and
collaborator invite alike, so one Remove was the whole teardown.
