# Phase 261 - The Test That Found Nothing to Break

We installed the panel on a blank virtual machine, set it up the way a customer would, and then tested it
from the outside with nothing but a web browser's worth of access.

## What held

The core protections behaved as intended: login handling, form protection, session handling, access
between accounts, and the setup flow.

## What we tightened

Nothing let us in, but a handful of small things were more informative to an outsider than they needed to
be, and one protective measure was broader than intended. All of them are fixed in v0.20.10, and the
detail lives in the published report rather than here.

## One we left alone on purpose

One setting was kept as it is because tightening it would have broken a legitimate sign-in flow, and the
existing protections already cover the risk. A tighter setting that breaks sign-in is not an improvement.
