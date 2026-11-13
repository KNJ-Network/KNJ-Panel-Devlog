# Phase 274 - The Redo Carries On

The page-by-page redo continued with the pages that had been left until last, and with one look we liked so
much we gave it to every page that could use it.

## What changed

**Package Manager.** The page was a list of boxes. It is now a catalogue of useful server software grouped
by what it is for: caching, images, video and documents, build tools, admin tools and security checks. Each
entry says what it does and what else it needs, for example that a cache also needs its extension from PHP
Manager. Every install is kept in a history with its log, and the install runs in a live terminal.

**Uninstalling, carefully.** You can now remove software from the same page, but only what you installed
from it. The server keeps its own record of that, so a package that was there before, or that the panel
itself depends on, cannot be removed from here. The page shows what a removal would take with it before it
does anything.

**PHP Workers.** A busy site can now be given more PHP workers than the default. Administrators choose, per
account, how workers are managed and how many may run at once. Each setting carries a plain warning, and the
page shows how much of the server's memory the workers could use, so a generous setting is a decision rather
than a surprise. The settings survive a change of PHP version, which used to reset them.

**Webmail sign-in.** It now matches the panel's own sign-in pages, with the version shown at the bottom.

**One terminal look.** The live terminal from the Package Manager, a dark card with a title, a small status
pill and the log, is now used everywhere the panel shows a running job or command output. That is the
installers, updates, scans, migrations, imports, deploys, logs, diagnostics, DNS lookups and the firewall
page: twenty-two pages that used to each do it slightly differently.
