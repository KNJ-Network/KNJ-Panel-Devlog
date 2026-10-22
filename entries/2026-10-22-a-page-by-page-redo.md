# Phase 270 - A Page-by-Page Redo

The new look in 0.21.0 gave every page a fresh header, but the pages themselves were still laid out the way
they had been built, one feature at a time. So we went through the panel one page at a time, drew several
options for each, and rebuilt the ones that earned it.

## What changed

**Packages and resellers.** A reseller now has a plan that sizes their whole account, their own website and
every customer they host together, and they can build their own packages inside it. You decide whether they
sell only their own packages, those plus ones you pick, or those plus all of yours.

**Email Customers.** The old broadcast page became a window on the account list: choose everyone, chosen
accounts, a package or your resellers, write in a proper editor, send yourself a test, and see everything
you have sent.

**Migration.** One page for every way to bring an account in. An uploaded backup is read first and shown to
you, with a checklist, and nothing is created until you press Restore.

**Server Setup and Servers.** A new server opens on five short guided steps. Once it is set up, the page is
a settings view with a few things to finish, and Servers became a set of health cards: memory, disk, load,
uptime, updates waiting and whether a restart is needed.

**Notifications.** Every alert is now one line with an email switch and a tag for each Slack, Discord or
custom channel, so you choose per alert where it goes. There are 28 alerts, including new checks for a full
disk, low memory, a stopped service, a backed-up mail queue, an overdue backup and a certificate close to
expiry. A problem that carries on alerts once, not every few minutes.

**Licence.** The page shows how far through the trial you are, what a licence covers and the price, and
for a paid licence who it is registered to and the key, masked until you ask to see it.

## Still to come

Resellers get their own Notifications page next, so they hear about their own customers.
