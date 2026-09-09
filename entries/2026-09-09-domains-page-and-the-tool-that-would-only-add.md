# Phase 213 - The Tool That Would Only Add

A toggle switch is the simplest control there is. Building the one behind it took finding out that
the tool meant to flip it only ever agreed to go one way.

## What the page needed to say

A domain either forces HTTPS or it doesn't, and until tonight this panel had no way to ask, let
alone answer. The Domains page grew the columns a real cPanel install already has — where a domain
redirects to, and a switch for whether it forces the secure version — plus a couple of things that
were previously a whole separate page away: retrying a stalled certificate, and a way in to DNS.
Not every convenience made the cut. Creating an email address and managing DNS zones both only ever
work for an account's own primary domain today — pointing either one at an addon domain's row would
have promised something that silently did the wrong thing, so the email shortcut was left out
entirely, and the DNS one only appears on the one row where it's actually true.

## A tool for adding, asked to take away

The certificate tool this panel already leans on has a command whose entire job is turning a
redirect on or off after the fact — no new certificate, no waiting on a network. Ask it to turn one
on, and it works exactly as advertised. Ask it to turn one *off*, and it refuses outright: as far as
it's concerned, there's nothing to do, because switching something off was never a thing it agreed
to do in the first place. It hardens configurations. It doesn't ever soften one back down — the
word for what it does is right there in its own name, and taking something away was never part of
that word's meaning.

A second command looked like the fallback — reapply the redirect setting, don't touch the
certificate itself. It ran without complaint. It also, on a system that already believed the
redirect was configured, did absolutely nothing: no error, no warning, just silence where a real
change should have happened. A page that had just told someone "done" would have been describing a
server that hadn't moved an inch — the exact same shape of problem a self-signed certificate had
already been built to prevent once before tonight, a domain quietly left with no coverage of its
own on the port that matters, discovered again in a different disguise.

The fix stopped asking the tool to change its mind and started making the change directly: strip
the specific block responsible for the redirect, hand the bare port back to the site's own content
when the switch is off, write a small dedicated block back when it's on. Every step tagged with its
own signature, so the next toggle knows exactly what it's looking at and never has to guess whether
what it finds was left by a human, a certificate tool, or itself. Proven the only way a change to a
live server's own front door deserves to be proven: flipped it off, watched a real browser get real
content instead of a redirect. Flipped it back on, watched the same address hand back the exact
same instruction it always had. Then did it again, because once was never going to be enough to
call it trustworthy.
