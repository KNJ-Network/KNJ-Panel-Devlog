# Phase 73 - A Real Home Screen for the Account Panel

The account dashboard was always the thinnest page in the whole panel — four stat tiles and a
"your site" card, nothing else. Everything else lived one click away behind the sidebar. Today
that changed: logging in now shows a proper home screen.

## What's on it now

- **Account status, primary domain, disk usage, and bandwidth** up top, each with a real number
  pulled from the account's own usage tracking, not a mockup — the same disk/bandwidth figures
  the account's package quotas are checked against everywhere else in the panel.
- **Account and server information** underneath — username, home directory, server hostname, and
  shared IP, the handful of facts a website owner occasionally needs to hand to a developer or
  paste into a support ticket.
- **Every section the sidebar has, as its own card**, and every feature inside that section listed
  underneath it — Files, Domains, Email, Databases, Metrics, Security, Software, Advanced,
  Preferences. Each feature gets a one-line description of what it actually does, in plain
  language, not just a label. Clicking a card entry goes straight to that feature — the dashboard
  is a real front door, not a dead end you glance at once and never return to.

## A new icon set, built from scratch

Every feature on the dashboard needed its own icon, and pulling in a third-party icon library
felt wrong for something meant to carry the panel's own visual identity — so this became a small
project of its own: a hand-authored set of about 45 line icons, one per feature plus one per
section, built as a single shared component the rest of the panel can now reuse anywhere an icon
is needed.

## Keeping the sidebar and the dashboard honest with each other

The sidebar nav and the new dashboard cards both need the exact same list of features, with the
exact same permission and package-feature gating — a collaborator who can't see the Security
group in the sidebar shouldn't see it as a card either, and a package that doesn't include FTP
shouldn't offer it from either surface. Rather than keep two separate lists of the same ~45
features in sync by hand, both now render off one shared catalog. Add a feature once and it shows
up correctly in both places, gated the same way, automatically.

## Next

Back to the click-through comparison against the wider hosting-panel feature set — a few genuine
gaps turned up (calendar/contact sync, a richer BoxTrapper-style spam quarantine, per-file backup
restoration, and a couple of smaller ones) that are still just findings for now, not yet built.
