# Phase 03 - A Real Course Correction

Worth being honest about this one rather than quietly editing history: earlier planning treated
"multi-server" as generic web-hosting-node clustering — accounts distributed across an
interchangeable pool of machines. That's not the shape a real hosting stack needs.

The right model is a self-contained full stack on one box — web, mail, database, and DNS all
together. The one genuinely useful cross-server relationship is DNS clustering: a main server can
push its DNS out to dedicated DNS-only servers. No fleet of interchangeable web nodes.

Corrected the model to match:

- **Main server** — the full stack. Every account and site lives here. This is what the dev box
  already is.
- **DNS-only server** — a stripped-down install of the same software, with DNS secondary functions
  only, syncing from the Main server.

The knock-on effect: the core provisioning milestone (create an account, get a real site) turns
out not to need a remote component at all — it's local execution on the Main server. A dev VM that
had already been stood up for the wrong reason got renamed and repurposed rather than wasted.

Not the most exciting entry, but getting the foundation right before more gets built on it
matters more than looking tidy.
