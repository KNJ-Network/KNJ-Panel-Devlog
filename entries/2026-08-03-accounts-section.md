# Phase 47 - Finishing Accounts, and Finding Two That Were Already Done

Six items were sitting in the Accounts section of the roadmap: edit account, bulk account removal,
admin password reset, package upgrade/downgrade, broadcast messaging, and access log downloads.
Before writing new code, the actual codebase was checked rather than trusting the roadmap's status.
Two of the six turned out to be already fully built: editing an account's owner name, email,
package, and reseller assignment, and changing the package, which already re-applies the new disk
quota immediately. That's the difference between building something twice and just fixing a stale
status.

The four genuinely new items shipped together. Bulk account removal turned out to be mostly a UI
problem: the select-all/per-row checkbox pattern built for KNJ Webmail's bulk actions dropped onto
List Accounts almost unchanged, and single and bulk removal now run through identical cleanup.
Each account still gets its own background removal job.

Admin password reset reused the choice account creation already offers — auto-generate (shown once)
or set explicitly — on a full page rather than an inline row action.

Access log downloads reused the account panel's existing Raw Logs page logic, with the admin side
just needing a different way of choosing which account to look at.

Broadcast messaging reuses the notification pattern built for two-factor login codes and the
existing background queue. The one deliberate choice was scope: every account owner on the server,
matching the roadmap's wording rather than narrowing it. Verified with a throwaway test account and
a real broadcast, confirmed in the mail log with the right recipient, subject, and body.

Every item got its own commit, tests, and live check on a development server. All six rows are
marked Live now, including the two that were already there.
