# Phase 171 - A Firewall Page That Actually Does Something

The Firewall page under Controller → Security was largely a status display with unban buttons. We replaced it with a structured, editable rule table.

Because mistakes here can lock an admin out of their own server, we planned this carefully: researched the existing code, verified real firewall output on a development server, and reviewed the first design for failure modes before writing code.

## Design decisions

- General rules are kept separate from Access Control, which has a different data model and sync approach.
- Each rule is identified by its owner, so the page can show general rules as editable and show everything else, including the install baseline, as read-only.
- General rules cannot touch ports already managed by the install baseline or Access Control, which prevents rules from conflicting and prevents accidental self-lockout.
- Ordering is kept predictable so rules cannot undermine each other.
- Reading firewall state goes through the existing provisioning script, so the feature works on upgraded servers as well as new installs.

## Verifying

2638/2638 passing, with 28 new tests built against real captured firewall output. On the development server we added, verified and removed a test rule through the UI, confirmed that a rule on a protected port is rejected before reaching the script, and confirmed administrative access stayed available throughout.

Tested (2638/2638).
