# Phase 14 - A Second Server, a Real Install Script, and Multi-PHP Finally Working

Multi-PHP version support — installing several PHP versions side by side, letting each account
pick its own — has been "next" for a while. The first real attempt hit a wall immediately: the
original dev server's OS was too new for the PPA that provides multiple packaged PHP versions on
Debian/Ubuntu. Rather than fight that, we stood up a second server on an OS generation old enough
to actually support it, and treated the move as a forcing function for something that had been
owed for a while anyway: a real, unattended install script.

## Why a second server, not a workaround

The standing rule going into this: **the panel has to be able to do everything it needs to do
without an operator ever running a command over SSH.** That's not just a nice-to-have — it's the
actual product requirement for anyone who isn't us installing this on their own box. Patching
around a missing PPA on the original server would have meant hand-installing PHP versions by
hand, which fails that requirement outright. A second server on a properly-supported OS, paired
with a script that does the whole install from a bare OS, is the version of this that actually
generalizes.

## What the install script does

`bootstrap-server.sh` runs once, on a freshly-installed OS, and gets a fully working panel out
the other end — web server, database, mail server, nameserver, firewall, intrusion prevention,
the panel's own dedicated service account, and the app itself, checked out from git and running.
No manual step in the middle. It reached that point after nine real bugs, each one found by
actually running it against the live server rather than reasoning about it in the abstract — the
most persistent shape of bug being permission checks that quietly return "doesn't exist" instead
of "access denied" when they test a root-owned path as the wrong user. Same failure mode, five
separate places, before it stopped showing up.

One deliberate architectural change came out of this too: the app now runs as its own dedicated,
unprivileged system account rather than under whichever person's login happened to set the server
up. That was flagged as follow-up work in the last security audit — a passwordless sudo grant on
the same account the web app runs as. Now the two identities are genuinely separate: the app's
account has exactly the three narrow sudo rules it needs, and nothing more.

## Multi-PHP, actually working

With a properly-supported OS underneath it, the feature this whole detour was in service of
finally works: install a PHP version from the panel, watch it build with a live progress log, and
switch an individual account onto it. Two more pieces went in alongside it, both meant to feel
like moving between panels shouldn't require memorizing which is which:

- An **account-side PHP INI Editor** — the customer-facing half of PHP configuration, separate
  from the admin-side version manager, matching the Controller/Account split already used
  throughout this panel.
- A **PHP module/extension manager** — toggle optional PHP extensions per version, plus one-click
  ionCube Loader installation for the (still surprisingly common) legacy commercial software that
  requires it.

## What's next

The install script currently assumes the operator already knows the domain they're pointing at
the server, which isn't realistic for a normal install where DNS isn't set up yet. That's the
next thing to fix — deferring the real TLS certificate to a first-run setup step instead of
demanding it upfront.
