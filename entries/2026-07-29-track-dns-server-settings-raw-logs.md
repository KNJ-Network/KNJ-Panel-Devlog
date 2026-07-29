# Phase 30 - Track DNS, Server Settings, and Raw Log Download

Three utility features boxed off in one session.

**Track DNS** lives under the DNS section of the WHM panel, accessible to both admins and resellers.
Type a hostname, pick a record type from the dropdown (A/AAAA/MX/TXT/NS/CNAME/SOA/PTR/SRV/CAA), and
it runs a live `dig` against the public DNS infrastructure — not the local cache — and shows the raw
answer section. The hostname is allowlisted to letters, digits, dots, hyphens, and underscores before
the shell call goes out.

**Server Settings** is a new WHM page (admin-only, under the Servers section) with two forms on it.
The top one sets the OS timezone via `timedatectl set-timezone` — full IANA timezone list in a
dropdown. The bottom one sets the OS-level server hostname via `hostnamectl set-hostname`, and also
reloads postfix with the new `myhostname` so mail starts identifying correctly immediately, without
needing a service restart. This is separate from the panel's own SSL hostname (which lives on the
Server Setup page and requires DNS to be pointing at the server before Let's Encrypt can issue).
The Server Setup page now shows a "fresh install?" callout with a direct link to Server Settings,
covering the post-install flow the user asked about.

**Raw Log Download** is on the account side — a "Raw Logs" page under Advanced lets an account
download their nginx access or error log as a plain-text file. The nginx vhost templates were updated
to write per-domain log files at `/var/log/nginx/{domain}.access.log` and
`/var/log/nginx/{domain}.error.log`. Existing accounts provisioned before this change will see a
"not available yet" notice until they're re-provisioned; new accounts get them automatically.
The download endpoint uses two new provision-script actions (`log-exists` and `read-log`) that
validate the domain against the same regex used everywhere else before constructing any path.

All three go through the existing provision script (`/usr/local/bin/knjpanel-provision-account`),
which already has a narrowly-scoped sudoers rule covering it.
