# Phase 73 - A Real Home Screen for the Account Panel

The account dashboard was the thinnest page in the panel: four stat tiles and a "your site" card,
with everything else a click away behind the sidebar. Logging in now shows a proper home screen.

## What's on it now

- **Account status, primary domain, disk usage, and bandwidth** up top, each pulled from the
  account's own usage tracking, the same figures the package quotas are checked against.
- **Account and server information** underneath: username, home directory, server hostname, and
  shared IP, the facts a website owner occasionally needs to hand to a developer or paste into a
  support ticket.
- **Every sidebar section as its own card**, with each feature listed underneath and a one-line
  plain-language description. Clicking an entry goes straight to that feature.

## A new icon set, built from scratch

Every feature needed its own icon, and pulling in a third-party icon library felt wrong for
something meant to carry the panel's own visual identity. So this became a small project of its
own: a hand-authored set of about 45 line icons, built as a single shared component the rest of
the panel can reuse.

## Keeping the sidebar and the dashboard consistent

The sidebar and the dashboard cards need the same list of features with the same permission and
package-feature gating. A collaborator who can't see the Security group in the sidebar shouldn't
see it as a card either. Rather than keep two lists in sync by hand, both now render from one
shared catalog.
