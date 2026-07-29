# Phase 30 - Track DNS, Server Settings, and Raw Log Download

Three utility features boxed off together.

**Track DNS** lives under the DNS section of the Controller panel, accessible to both admins and
resellers. Type a hostname, pick a record type from the dropdown (A/AAAA/MX/TXT/NS/CNAME/SOA/PTR/
SRV/CAA), and it runs a live lookup against public DNS infrastructure — not the local cache — and
shows the raw answer. Input is validated before the lookup runs.

**Server Settings** is a new Controller page (admin-only, under the Servers section) with two
forms. The top one sets the OS timezone from a full timezone list. The bottom one sets the
server hostname, and updates the mail server so mail starts identifying correctly immediately. This
is separate from the panel's own SSL hostname on the Server Setup page, which requires DNS to point
at the server before Let's Encrypt can issue. The Server Setup page now shows a "fresh install?"
callout linking to Server Settings.

**Raw Log Download** is on the account side — a "Raw Logs" page under Advanced lets an account
download its web access or error log as a plain-text file. Web server templates now write separate
per-domain logs. Accounts provisioned before this change see a "not available yet" notice until
they're re-provisioned; new accounts get them automatically.

All three go through the panel's existing audited, narrowly-scoped privileged path.
