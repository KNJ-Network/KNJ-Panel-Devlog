# Phase 58 - The Admin Roadmap, Verified Against the Real Code

Every admin roadmap item claimed Live at this point — the question was whether that was still
true, not just once when each section shipped. Four parallel passes, each checking a couple of
sections against the actual running code rather than trusting its status label.

Three real findings came back. Mailboxes & forwarders overstated itself — quota control reads as
per-mailbox but is actually one server-wide default set on Mail Settings, applied at creation
time; the wording's fixed to say that plainly. A genuinely built tool turned up with no roadmap
entry at all: Repair mailbox permissions, a one-click fix for ownership/permission drift on a
domain's mail storage, now listed. And Per-domain mail routing and Domain forwarding, both marked
Live on the client side since July, needed a scope caveat — self-service today only reaches an
account's primary domain; an admin can already do either for any other domain from the Controller
side, but the account owner couldn't tell that from the roadmap wording alone.

Everything else came back clean — real, working, matching its status. That closes the full admin
roadmap out for real, not just on paper. Next: the client side, starting with Email.
