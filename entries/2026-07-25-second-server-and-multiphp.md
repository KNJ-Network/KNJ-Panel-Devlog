# Phase 14 - A Second Server, a Real Install Script, and Multi-PHP Finally Working

Multi-PHP version support — installing several PHP versions side by side, letting each account
pick its own — has been "next" for a while. The first real attempt hit a wall: the original dev
server's OS was too new for the package source that provides multiple PHP versions. Rather than
fight that, we stood up a second server on an older OS generation that supports it, and treated the
move as a chance to build something that had been owed anyway: a real, unattended install script.

## Why a second server, not a workaround

The standing rule: **the panel should be able to do what it needs to do without an operator
running commands over SSH.** That's a product requirement for anyone installing this on their own
box. Hand-installing PHP versions would fail that requirement. A second server on a supported OS,
paired with a script that does the whole install from a bare OS, is the version that generalizes.

## What the install script does

The install script runs once on a freshly-installed OS and gets a fully working panel out the other
end — web server, database, mail server, nameserver, firewall, intrusion prevention, the panel's
own dedicated service account, and the app itself. No manual step in the middle. It reached that
point after nine bugs, each found by running it against a live development server rather than
reasoning about it.

One deliberate change came out of this too: the app now runs as its own dedicated, unprivileged
system account rather than under whichever person's login set the server up, with only the narrow
permissions it needs.

## Multi-PHP, actually working

With a supported OS underneath, the feature this detour was in service of works: install a PHP
version from the panel, watch it build with a live progress log, and switch an individual account
onto it. Two more pieces went in alongside it:

- An **account-side PHP INI Editor** — the customer-facing half of PHP configuration, separate
  from the admin-side version manager, matching the Controller/Account split used throughout.
- A **PHP module/extension manager** — toggle optional PHP extensions per version, plus one-click
  ionCube Loader installation for legacy commercial software that requires it.
