# Phase 58 - The Admin Roadmap, Verified Against the Real Code

Every admin roadmap item was marked Live at this point. The question was whether that was still
true, not just when each section shipped. Four parallel passes each checked a couple of
sections against the running code rather than trusting the status label.

Three findings came back. Mailboxes & forwarders described quota control more precisely than it
behaved, so the wording was corrected to say plainly how it works. A built tool, Repair mailbox
permissions, had no roadmap entry and is now listed. And Per-domain mail routing and Domain
forwarding needed a scope note: self-service reaches an account's primary domain, while an admin
can already do either for any other domain from the Controller side.

Everything else matched its status. That closes the admin roadmap review.
