# Phase 306 - Is The Server Coping

Resource Monitor was meant to be the first place to look when a server feels slow, and it was close to useless for that. It
listed every process sorted by a processor figure that was an average over the process's whole life, so something that had
just gone wrong could look quiet while the page's own check topped the list. Processes were raw commands, nothing tied a
process to the customer causing it, there was no history to say when a problem began, and the only action was a button that
killed a process on the spot.

## What changed

The page now says whether the server is coping before it lists anything: four numbers with the last day behind them, and a
short list of what needs a look, such as one customer using a large part of the processor for ten minutes. A chart goes back
as far as 30 days and says when the load stepped up and who started using more. Processes are named in plain words and tied to
the customer they belong to, and there is a view of every customer by how much they use.

Stopping something is deliberately narrower and gentler: only a customer's own programs, asked to finish first, and the
server's own services are shown as protected with a pointer to where they are restarted. Linked servers get their own tab.
