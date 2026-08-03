# Phase 47 - Finishing Accounts, and Finding Two That Were Already Done

Six items were sitting in the Accounts section of the roadmap: edit account, bulk account
removal, admin password reset, package upgrade/downgrade, broadcast messaging, and access log
downloads. Before writing a word of new code, the actual codebase got checked directly rather than
trusting the roadmap's own copy — the same discipline the Service Configuration batch just proved
out, run in the opposite direction this time. Two of the six turned out to already be fully built:
editing an account's owner name, email, package, and reseller assignment has been live for a
while, and changing the package already re-applies the new disk quota immediately. Neither one had
ever made it onto the public roadmap. That's not a wasted afternoon of investigation — it's the
difference between building something twice and just fixing a stale status pill.

The four genuinely new items all shipped together. Bulk account removal turned out to be mostly a
UI problem, not a logic one: the same select-all/per-row checkbox pattern already built for KNJ
Webmail's bulk delete/move dropped onto List Accounts almost unchanged, and the single-account
delete path got split into a small shared helper so both the one-at-a-time and bulk actions run
through identical cleanup rather than two versions of the same logic drifting apart later. Each
account still gets its own background deprovisioning job — bulk here means "loop and dispatch
several," not "teach the privileged script to handle a list."

Admin password reset didn't need a new pattern at all — account creation already offers a choice
between auto-generating a password (shown once, copy button included) or setting one explicitly,
and resetting an existing owner's password is exactly the same choice at a different point in
time. The only real decision was where it lives: a full page rather than an inline row action,
since a password-and-confirmation pair doesn't fit cleanly next to Suspend and Remove without a
modal, and this app doesn't use one.

Access log downloads was the closest thing to a copy-paste job, deliberately. The account panel
already has its own Raw Logs page, going through the privileged script's log-exists/read-log
actions rather than reading files directly from a web request. Those actions were already
domain-scoped rather than tied to whoever was asking, so the admin-side version just needed a
different way of finding out which account to look at — a route parameter instead of "whichever
account the logged-in user owns" — and nothing on the privileged side changed at all.

Broadcast messaging was the one place a wrong assumption would have mattered. It reuses the same
small Notification pattern already built for two-factor login codes, which meant no new
infrastructure — Laravel's own queue already picks it up, the same worker that's been running
since login security shipped. The one deliberate choice was scope: every account owner on the
server, not just the ones a given reseller manages, matching the roadmap's own wording rather than
narrowing it to something more convenient to build. Verified for real before calling it done — a
throwaway test account, a real broadcast sent through the actual queue, and the rendered email
confirmed sitting in the mail log with the right recipient, subject, and body, not just a green
test suite.

Every item got its own commit, its own test coverage, and its own live check against the real
running server before moving to the next one. All six rows on the roadmap are marked Live now,
including the two that turned out to already be there.
