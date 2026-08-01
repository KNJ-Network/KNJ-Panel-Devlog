# Phase 40 - Licensing, and Where the Line Sits

Fourth and final of the four v1.0 items. With this one shipped, all four are live and the freeze on
new feature work is lifted.

Licensing lives on its own separate server rather than inside the panel itself — a hardened
surface that a customer's install talks to over the network rather than something bundled into
what ships on their box.

**Trials are issued automatically, not requested.** A fresh install contacts the licence server
once during setup and gets back a 30-day trial with no admin action needed. The trial is tied to the
installation in a way that survives a reinstall, so a reinstall doesn't restart it.

**The real design work was deciding what a lapsed licence is allowed to do.** The obvious answer —
lock the customer out — is risky if not scoped carefully. This panel manages real websites, real
mail, and real DNS that other people depend on, and a licensing check that touched those would
punish a customer's own customers for a billing problem they have nothing to do with. So the rule
became: a lapsed licence can lock the panel's own front door, and nothing behind it.

That produced two tiers, and the second turned out narrower than expected. The first is immediate —
no valid licence, no access to the admin or account areas, with a status page explaining why and a
way to activate a key. The second was meant to pause the panel's own automation after a further
week, but cron jobs and certificate renewal turned out not to be the panel's to pause, since they
run through the operating system. Tier two ended up being just the panel's own scheduled backup
run. Everything else a customer built keeps running regardless.

Live-verified end to end on a development server: a fresh trial issued with a countdown banner,
expiry simulated and the lockout firing immediately, a key entered on the status page, and full
access back as soon as it was accepted.
