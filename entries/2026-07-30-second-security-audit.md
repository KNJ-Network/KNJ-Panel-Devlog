# Phase 34 - The Second Full Security Audit

Five days and a large number of commits since the first one — the Security Center, Packages &
Resellers, a custom-built webmail client, account creation and suspension, and today's disk quota
and bandwidth work all shipped in that window. Time to step back and look at everything as a whole
again.

## What it covered

Same shape as the first audit, scaled up: a full re-read of every privileged action, every
account-side page that takes a resource ID re-checked for ownership enforcement, server file
permissions and network exposure, database privileges, and the full commit history of all our
repositories for anything that should never have been committed.

The webmail client got the deepest look, since it's the first part of the app that renders content
someone outside the panel controls — an inbound email. Compose sanitization, attachment handling,
the mail-filtering-rule generator, and CSV contact import were all checked for the relevant
injection classes.

New this time: adversarial, not just read-only, verification of account isolation, on a development
server, including confirming that a write past a quota limit is blocked at the operating-system
level rather than just measured.

## What it found

Most of it held up — the webmail client's handling of untrusted email content, account isolation,
and the new disk quota work all passed live testing. Two smaller items turned up, both fixed the
same day:

- A hardening setting that had been configured correctly wasn't taking effect in one respect.
  It was fixed, and a check now verifies the real, effective outcome rather than only that the
  setting was written.
- One input to a privileged action wasn't validated as strictly as its siblings. It wasn't reachable
  from the app, but was closed anyway and verified with hostile input.

Nothing here was a case of someone getting in — this was proactive, not incident response.

## What's next

These audits keep running regularly while the panel is under active development.
