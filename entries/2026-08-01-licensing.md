# Phase 40 - Licensing, and the Line It Can Never Cross

Fourth and final of the four v1.0 dealbreakers. With this one shipped, all four are live — the
freeze on new feature work is lifted.

Licensing lives on its own separate server, not inside the panel itself, the same posture as the
build/backoffice infrastructure — a hardened, admin-only surface that a customer's install talks to
over the network rather than something bundled into what ships on their box.

**Trials are issued automatically, not requested.** A fresh install calls out to the licence server
once during setup and gets back a 30-day trial with no admin action needed. The interesting part is
what that trial is keyed to: not the OS's own machine identifier, which a reinstall regenerates, but
a hardware-level one that survives wiping the box and starting over. That's deliberate — it's the
one thing standing between "30-day trial" and "unlimited trial, just reinstall when it runs out."

**The real design work was deciding what a lapsed licence is allowed to do.** The obvious answer —
lock the customer out — is also the dangerous one if it's not scoped carefully. This panel doesn't
host anything itself; it manages real websites, real mail, real DNS that other people depend on. A
licensing check that ever touched those would be punishing a customer's own customers for a billing
problem they have nothing to do with, and there's no version of that which doesn't end badly for
everyone's trust in the product. So the rule became absolute: a lapsed licence can lock the panel's
own front door, and nothing behind it.

That produced two tiers, and the second one turned out narrower than expected once actually
building it. The first tier is immediate — no valid licence, no access to the admin or account
areas, full stop, with a status page explaining why and a way to activate a real key on the spot.
The second tier was meant to also pause "the panel's own automation" after a further week — cron
jobs, certificate renewal — as a softer, slower consequence short of a hard lockout. Turned out
neither of those is actually the panel's to pause: cron jobs run as real per-account crontab
entries installed straight into the OS, outside this app entirely, and certificate renewal is
handled by its own independent system timer. Both are squarely "the customer's own stuff running,"
exactly what the rule already said to leave alone — so tier two ended up being exactly one thing:
the panel's own scheduled backup run. Everything else a customer built keeps running regardless,
by construction, not by carve-out.

Live-verified end to end, watched in real time rather than just trusted from a green test suite: a
fresh trial issued and showing a real countdown banner, the expiry simulated and the lockout firing
immediately, a real key entered on the resulting status page, and full access back the instant it
was accepted — no reload, no delay, no manual step in between.
