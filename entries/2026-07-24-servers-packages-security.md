# Phase 02 - Servers, Packages & Security Tooling

Second milestone done the same day as the first — the controller (WHM-equivalent) area now
does real work rather than just existing as a shell.

## What's working

- **Servers**: register hosting nodes (name, hostname, IP, role — web/mail/DNS-only/controller),
  full create/edit/remove. The dev box has registered itself as the controller.
- **Packages**: resource templates — disk quota, bandwidth, email/database/addon-domain limits —
  full CRUD, ready to be assigned to accounts once account provisioning exists.
- **Firewall**: a real status page over the server's actual `ufw`/`fail2ban`, not a mockup. While
  building this, fail2ban had already banned a real IP from live internet scanning traffic
  against the public dev box — good, unplanned proof the whole security stack from the last
  session is actually doing its job.
- **API tokens**, **backup configuration**, and a **service status monitor** (Nginx, PHP-FPM,
  MariaDB, fail2ban, SSH, Docker, Tailscale — all reporting correctly).

## A security note worth mentioning

The firewall/fail2ban page needs to run privileged commands from a web request. Rather than
give the app broad sudo access, it gets a narrowly scoped sudoers rule for exactly the specific
commands it needs — nothing else. Small thing, but it's the kind of default that matters a lot
more once real customer accounts exist.

## Next

M2: the node agent. Right now the controller only really manages itself — the next milestone is
teaching it to talk to *other* servers, which is what actually makes the multi-server model real.
