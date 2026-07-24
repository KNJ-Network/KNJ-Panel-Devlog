# Phase 02 - Servers, Packages & Security Tooling

Second milestone done the same day as the first — the Controller area now does real work rather
than just existing as a shell.

## What's working

- **Servers**: register hosting nodes (name, hostname, IP, role — web/mail/DNS-only/controller),
  full create/edit/remove. The dev box has registered itself as the controller.
- **Packages**: resource templates — disk quota, bandwidth, email/database/addon-domain limits —
  full CRUD, ready to be assigned to accounts once account provisioning exists.
- **Firewall**: a real status page over the server's actual firewall and intrusion-prevention
  tooling, not a mockup.
- **API tokens**, **backup configuration**, and a **service status monitor** for the main services,
  all reporting correctly.

## A security note worth mentioning

The firewall page needs to run privileged commands from a web request. Rather than give the app
broad access, it gets narrowly scoped permissions for exactly what it needs. Small thing, but the
kind of default that matters more once real customer accounts exist.
