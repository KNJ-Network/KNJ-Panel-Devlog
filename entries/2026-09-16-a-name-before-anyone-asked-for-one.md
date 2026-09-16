# Phase 238 - A Name Before Anyone Asked for One

The very first thing a brand new server does, before anyone has logged in, before anyone has
decided what to call it, is introduce itself. Until now, that introduction came with an asterisk.

## The honest handshake nobody wants

A fresh install had a real, working certificate to offer — just not one anyone's browser trusted,
because it was standing in for a name that didn't exist yet. Every visitor got the same small
warning: this might not be who it says it is. Technically true, and technically fine, since the
person seeing it was usually the very admin who'd just built the thing. But "technically fine" and
"the first thing you see actually looks broken" are two different experiences, and only one of them
is the one that matters on a first impression.

## Borrowing a name that was never really borrowed

The fix wasn't to invent a certificate trick. It was to give every new server a real name to answer
to before its owner ever chooses one — computed, not assigned, the instant it exists. Feed the
server's own address into a hostname, and a small dedicated service on the other end hands back
exactly that address again, every time, for any address that could ever exist. Nothing is stored.
Nothing is provisioned in advance. The answer was never sitting in a list waiting to be looked up —
it's worked out fresh from the question itself, which means there's no such thing as a server this
system doesn't already know how to answer for.

## Two places where "looks fixed" and "is fixed" quietly split apart

The build surfaced two of exactly that kind of gap — not wrong, just not yet true.

One: a setting got written to the right file, but the running program had already memorized an
older version of that file and had no reason to go back and check again. The edit was real. It
just wasn't listening.

Two: a piece of bookkeeping that runs itself every fifteen minutes, quietly reconciling what the
server believes about itself, turned out to be checking its own name against a name nobody had
actually given it yet. The record existed. The introduction hadn't been made.

Both were the same shape wearing different clothes — a correct change, made in the wrong place to
actually be felt yet. Caught by going back and asking, deliberately, "does this take effect right
now, or does it just look like it will eventually" — and in both cases, eventually wasn't good
enough.

## What actually finished

A server that's never met its owner yet still has something honest to say about itself the moment
it boots — a real name, a real certificate, no asterisk. The trust warning that used to be the
default first impression is now the fallback of last resort, reserved for the rare moment
everything else has already failed.
