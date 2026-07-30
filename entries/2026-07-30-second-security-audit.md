# Phase 34 - The Second Full Security Audit

Five days and about 160 commits since the first one — the Security Center, Packages & Resellers,
a full custom-built webmail client, account creation and suspension, and today's disk quota and
bandwidth work all shipped in that window. Time to step back and look at everything as a whole
again, not just the pieces tested individually as they landed.

## What it covered

Same shape as the first audit, scaled up: a full re-read of the one privileged script every
account-mutating action funnels through (61 distinct operations now, up from about 15), every
account-side page that takes a resource ID re-checked for real ownership enforcement, live server
file permissions and firewall/listening-port state, database privilege grants, and the full commit
history of all three of our repos for anything that should never have been committed.

The webmail client got the deepest look, since it's the first thing in this app that renders
content someone outside the panel fully controls — an inbound email. Compose sanitization,
attachment handling, the mail-filtering-rule script generator, and CSV contact import all got
checked specifically for the injection classes that matter for that kind of surface.

New this time: adversarial, not just read-only, verification for account isolation. Rather than
just reading the code and trusting it, one account's actual running process was used to try
reading another account's files, and a real file write past a quota limit was attempted to confirm
it's genuinely blocked at the operating-system level rather than just measured.

## What it found

Most of it held up cleanly — the webmail client's handling of untrusted email content, every
account's inability to see another account's files, and the new disk quota work all passed live
adversarial testing, not just a read-through. Two real things did turn up, both fixed the same day:

- A server hardening setting that every previous check had confirmed was *configured* correctly
  turned out not to be *taking effect*, due to how the operating system resolves conflicting
  configuration files loaded in a particular order — a subtle gap between "the file says the right
  thing" and "the file actually wins." Closed by fixing the precedence and adding a check that
  verifies the real, effective outcome going forward, not just that the setting was written down
  somewhere.
- One piece of input to the privileged provisioning script wasn't being validated as strictly as
  every other piece of input to it is — not reachable by anything in the app today, but exactly
  the kind of gap that's cheap to close now and expensive to regret later. Closed, and verified
  live that a deliberately malicious version of that input gets rejected with zero side effects.

Nothing here was a case of someone getting in — this remains proactive, not incident response.

## What's next

Same standing commitment as before: these keep running regularly while the panel is under active
development. We're building this in the open specifically so anyone considering it later can see
that security work isn't an afterthought bolted on before launch — it's been part of the process
since week one. A public summary of these audits (checks run, pass/fail by category, no code or
private details) is next up.
