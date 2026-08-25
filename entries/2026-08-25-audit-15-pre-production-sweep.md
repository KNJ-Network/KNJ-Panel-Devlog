# Phase 130 - Audit #15: A Toggle That Didn't Do What It Said

The first audit run explicitly as a readiness check ahead of a release, rather than only a post-feature
diff. Scope widened past the usual code review to include a compromise check of the servers in use.
It was reviewed by area across everything since Audit #14: the WAF, CalDAV/CardDAV, Subscribers and
Marketing Email, off-site backups, and the remaining mail-hardening features.

## The real finding

The antivirus setting's "on infected" value was passed to the provisioning script in a different letter
case than the scanner's configuration expects, so the script rejected every real save from the UI. The
setting itself was saved before the script ran, so the checkbox kept reading "enabled" even though the
scanner had not been configured.

Fixed on both ends: the value is converted only where it is written into the scanner's format, and the
setting is now stored only after the script call succeeds. Two new tests pin both halves.

## What else turned up

- **CalDAV/CardDAV failed logins were not logged** to the channel the intrusion-prevention jail
  watches. They now log to the same channel and format as webmail and panel logins. A regression
  test also confirms against the real DAV server that one mailbox cannot reach another's calendar.
- **CSV formula injection in the Subscribers export.** A name beginning with a spreadsheet formula
  character reached the exported CSV verbatim. Such values are now neutralized with a leading quote.
- **A campaign controller did not re-verify the route's list id** for some actions, which could show
  another list's name. It now performs the same ownership check as its sibling actions.
- **Three smaller findings:** the backup destination's encrypted field was missing the hidden-attribute
  marker used on other secret-bearing models; DNS zone import's apply step lacked the size cap its
  preview step has; and the public subscribe endpoint gained a per-recipient throttle on top of its
  per-IP one.

## Clean

The compromise check found no evidence of unauthorized access. The WAF's gating and fail-closed
behavior held. Off-site backups are encrypted at rest and never re-displayed. DNS zone import and
export cannot inject anything manual entry would reject. Subscribers' cross-account checks came back
clean. A secret scan across the repositories found nothing real.

## Verified

Full local suite (1,914 tests) and the code-style check green after every fix. Deployed to the dev
server, where the corrected scanner value and the new DAV login logging were confirmed live.
