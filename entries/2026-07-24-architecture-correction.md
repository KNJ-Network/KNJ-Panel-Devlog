# A Real Course Correction — 24 July 2026

Worth being honest about this one rather than quietly editing history: earlier planning treated
"multi-server" as generic web-hosting-node clustering — accounts distributed across an
interchangeable pool of machines. That's not actually how cPanel/WHM works, and it's not what
was wanted here either.

Real cPanel/WHM is a self-contained full stack on one box — web, mail, database, and DNS all
together. The only native cross-server relationship it has is DNS clustering: a main server can
push its DNS out to dedicated DNS-only slave servers. That's it. No fleet of interchangeable web
nodes.

Corrected the model to match:

- **Main server** — the full stack. Every account and site lives here. This is what the dev box
  already is.
- **DNS-only server** — a stripped-down install of the same software, DNS-slave functions only,
  syncing from the Main server.
- **Mail-only server** — a real idea, explicitly a separate future project, not part of this build.

The knock-on effect: the core provisioning milestone (create an account, get a real site) turns
out not to need a remote agent at all — it's local execution on the Main server, no second
machine involved. The node agent only actually needs to exist for DNS-only linking, much later
in the roadmap. A dev VM that had already been stood up for the wrong reason got renamed and
repurposed rather than wasted.

Not the most exciting entry, but getting the foundation right before more gets built on it
matters more than looking tidy.
