# Phase 10 - The First Full Security Audit

Every milestone so far has included testing its own feature against the live server — real
attack attempts against File Manager, real cross-account access attempts against the Database
Manager and DNS zone editor, a real check that the mail server isn't an open relay and the
nameserver isn't an open resolver. That's all been testing one feature at a time, though. This
was the first pass that stepped back and looked at everything built so far as a whole: every
privileged operation, every place one account's data could theoretically leak into another's,
the actual live server's exposed ports, logs, and file permissions, and the full history of both
repos for anything that should never have been committed.

## What it covered

- Re-read of the one privileged script every account-mutating action in the whole panel funnels
  through, checking every action's input validation and injection surface, not just the ones
  touched most recently.
- Every account-side and admin-side page that takes a resource ID re-checked for "does this
  actually confirm the resource belongs to the person asking," not just the ones added in the
  most recent milestone.
- The route table, session/auth configuration, live server file permissions, database privilege
  grants, firewall rules and listening ports, fail2ban and auth logs, and full git history of
  both repos.

## What it found

Most of it came back clean — the account-isolation checks built up milestone over milestone held
up, and there's no leaked secret anywhere in either repo's history. A few real things did turn
up, all fixed the same day:

- A handful of places wrote a short-lived secret (a password, an uploaded certificate's private
  key) to a temp file and locked down its permissions a moment *after* creating it rather than
  in the same step — a narrow window where the file was more exposed than it should've been.
  Closed by making the restrictive permissions part of file creation itself, not a follow-up step.
- A server configuration file had slightly looser file permissions than it needed.
- A couple of session cookie security settings were left to defaults instead of being set
  explicitly, given this panel is HTTPS-only.
- One overly broad administrative permission on the server that should have been scoped down
  alongside the narrower, purpose-specific rules already sitting next to it.

Nothing here was a case of someone getting in — this was proactive, not incident response. It's
the kind of thing that's easy to let slide by while moving fast on features, which is exactly why
this is becoming a standing part of the process now rather than a one-off.

## What's next

These audits will run frequently while the panel is still under active development — most weeks,
sometimes more — then taper to a regular weekly or monthly cadence once the build reaches a
stable, feature-complete state. The full detailed findings stay in the private engineering log;
this is the shape of what gets shared here going forward.
