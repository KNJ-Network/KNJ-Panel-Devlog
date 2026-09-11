# Phase 228 - A Name Borrowed From Someone Else's House

A subdomain in this panel has never owned its own patch of DNS. It's always lived inside its
parent's — a name recorded in the same file, answered by the same nameserver, never delegated
anywhere of its own. That arrangement has one real consequence that hadn't come up until tonight:
a feature that writes "this domain's" records has to be told, explicitly, whether "this domain"
means the file it's editing or just a name inside it.

## The check that only knew how to check itself

A subdomain's SPF and DKIM records had a real gap: nobody had ever written any, because nothing
had ever offered to. The repair button that existed for every other domain simply didn't appear
for a subdomain at all — not broken, just never built for a shape of domain that had no zone of
its own to repair. Confirmed directly: this account really does send mail from a subdomain, so the
gap wasn't theoretical.

## Two protocols, two different rules

DMARC turned out to be its own separate question, not the same gap wearing a different name. SPF
and DKIM are strict about exact matches — if you're going to send mail as a name, that name needs
its own record, full stop. DMARC was deliberately designed with an escape hatch: a name with no
policy of its own inherits its parent's, the same way a piece of unwritten paperwork defers to
whatever the file above it already says. A subdomain missing its own DMARC record isn't
unprotected. It's covered, just not by anything filed under its own name.

Treating both cases the same way — "missing means broken" — would have been wrong for one of them
and quietly correct for the other, purely by coincidence. Getting this right meant asking the two
protocols different questions, not applying one rule to both and hoping it landed right twice.

## Writing into a file that isn't yours

Once the subdomain case was properly told apart from the ordinary one, actually writing a
subdomain's own records meant editing the parent's file at a name that isn't the parent's own —
adding an entry labeled for someone else's corner of the house while still living under the same
roof. The mechanism that already knew how to write a domain's mail records had only ever been
asked to write at the top of the file. Teaching it to write at an interior name instead — and to
generate a signing key that belongs to that name specifically, never borrowed from the parent's —
turned "repair this domain" into something that actually means what it says regardless of which
kind of domain is asking.

## What actually finished

A subdomain that genuinely sends its own mail can now get its own real SPF and DKIM coverage, and
a subdomain that doesn't gets an honest DMARC answer instead of a false alarm borrowed from a
protocol that was never actually broken for it.
