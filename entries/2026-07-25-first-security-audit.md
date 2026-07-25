# Phase 10 - The First Full Security Audit

Every milestone so far has included testing its own feature against a live development server —
hostile input against File Manager, cross-account access attempts against the Database Manager
and DNS zone editor, a check that the mail server isn't an open relay and the nameserver isn't an
open resolver. That's all been testing one feature at a time, though. This was the first pass that
stepped back and looked at everything built so far as a whole, including the full history of the
repositories for anything that should never have been committed.

## What it covered

- Input validation and injection surface across every privileged action, not just the recently
  touched ones.
- Every page that takes a resource ID, re-checked for confirming that the resource belongs to
  the person asking.
- Session and auth configuration, file permissions, database grants, firewall rules, exposed
  services, and logs.

## What it found

Most of it came back clean: the account-isolation checks built up milestone over milestone held
up, and no secrets were found in repository history. A few smaller hardening items turned up, all
fixed the same day:

- Temporary files holding short-lived secrets are now created with restrictive permissions from
  the start.
- A server configuration file had looser permissions than needed.
- A couple of session cookie settings are now set explicitly rather than left to defaults.
- One broad administrative permission on the development server was narrowed.

Nothing here was a case of someone getting in — this was proactive, not incident response, and it
is becoming a standing part of the process.

## What's next

Audits will continue regularly while the panel is under active development.
