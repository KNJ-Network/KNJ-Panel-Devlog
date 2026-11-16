# Phase 276 - A Diagnostics Page Worth Opening

The Diagnostics page ran six commands and showed their raw text. It was accurate and nobody opened it. The panel
already knows a great deal about whether a server is well, so we rebuilt the page around that.

## What changed

**A health check.** One button runs everything an administrator would otherwise check by hand: that each service is
running and the web server's own configuration is valid, how full the disk and memory are, whether the hostname points
at the server, whether the server's reverse DNS is set (which decides whether its mail reaches inboxes or junk),
whether mail can leave at all, whether the clock is right, whether any certificate is about to run out, how much mail
is queued, whether the server's address is on a blocklist, and whether backups and updates are current. It says "3
things need a look, nothing is broken", and for each one it says what it means and links to the page where it is put
right. It runs by itself every hour, so the page opens on a recent answer. A check that cannot be made is shown as
needing a look, never as a pass.

**Tools.** The checks you would otherwise open a terminal for: ping, trace the route, whether a port on another
machine can be reached, what certificate another machine presents, what a mail server says when you connect to it,
whether an address is on a blocklist, the busiest processes, failed services and the latest errors. Each result is a
live terminal card you can copy or download, and every run is kept for thirty days.

**A support report.** One file with the versions, the health check, the state of every service, disk, memory and load,
and the latest errors, for when someone else needs to see what you are looking at. Passwords, keys, tokens, email
addresses and customer file names are left out, and addresses in the error logs are hidden.

## Why it stays read-only

Anything typed into a form becomes part of a command, so the tools accept only what a hostname, an address or a port
can be, and are checked again on the server itself, where it counts. We tested them with the kind of input an attacker
would try. The page reads and reports; it changes nothing.
