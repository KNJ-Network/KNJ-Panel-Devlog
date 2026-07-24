# Phase 04 - The Core Loop Actually Works

The milestone that proves this is a real hosting panel and not just an admin UI: creating an
account now provisions a real, isolated hosting environment. Not a mock, not a placeholder — a
genuine Linux user, an Nginx site, a dedicated PHP-FPM pool, and its own MySQL database, torn down
cleanly when the account is removed.

## What actually happens

Create an account through the controller UI, and in the background:

- A real Linux user gets created, with an isolated home directory
- An Nginx server block goes live for the domain
- A dedicated PHP-FPM pool, running as that account's own user
- A MySQL database and user, scoped to just that database

Verified by actually visiting the provisioned site and getting real PHP output back, not by
trusting a green checkmark.

## Two things worth being honest about

**A real bug, caught in our own testing**: provisioning initially ran synchronously inside the web
request, and a service reload it triggered interrupted the very request that started it. Fixed
properly: provisioning now runs as a background queued job with its own dedicated worker. A slow,
system-mutating operation has no business blocking a web request anyway.

**A feature deliberately not built yet**: File Manager was part of this milestone's original scope.
Direct file access needs careful protection against path traversal, and that deserves its own
focused pass rather than being bolted on at the end of an already-large milestone.

## Security shape

Privileged operations go through a single audited path with narrow permissions rather than a spray
of individually-permitted raw commands, and input is validated before anything privileged runs.
System usernames are generated internally rather than derived from user input.
