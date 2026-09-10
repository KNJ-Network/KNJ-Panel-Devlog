# Phase 214 - The Address That Worked the First Time

The last app taught the server a new rule: give every installed app its own address, kept current
automatically, never borrowed from whatever happens to sit at the site's own front door. The real
test of a lesson like that isn't whether it fixes the app that taught it. It's whether the next app
needs teaching at all.

## Nothing to fix this time

Invoice Ninja is the same shape as the app before it — no file called `login.php` anywhere, every
page a route decided by code, the exact shape that broke silently last time. Installed it, asked for
its own login page, and it just answered. Not "eventually, after a second fix" — the first time,
with nothing written specifically for this app. The rule learned once now applies to every app that
needs it, and this was the first chance to find out whether that was actually true or just true for
the one app it was built against.

## The install that wasn't where the obvious path led

The obvious way to install something built on the same framework as the last two apps is the way the
last two apps were installed: clone it, hand it to the dependency manager, let the framework's own
tools finish the job. Invoice Ninja's own documentation says not to do that — not as a suggestion,
as a warning against exactly that path for a real install.

The reason is that this app isn't really one thing. It's a framework's backend, sitting behind a
completely separate front-end application, built somewhere else entirely and never checked into the
same place as the code that serves it. Clone the backend and you get half an app — a server with
nothing to show anyone, and no simple command that builds the missing half for you, only a slow,
separate toolchain most of these servers have no reason to carry.

What actually works, once you stop assuming the obvious path is the real one: a single file, built
and published for exactly this purpose, with both halves already inside it — the backend's own
dependencies and the missing front end, already built, sitting next to each other. No extra
toolchain, no slow build step, nothing this server needs that it didn't already have. Slower to find
than the obvious wrong path. Faster, in the end, than every app installed before it.

## The password that had nowhere else to go

Somewhere in every one of these installs, a real admin account has to get created, with a real
password, without ever putting that password somewhere a stranger on the same machine could read it
back later. The apps installed so far all had a quiet way around that — a prompt that waits for the
password to be typed rather than handed over on the command line where anyone watching the process
list could see it.

This one doesn't. Its own account-creation command takes the password as a plain option and nothing
else — no prompt, no alternate door, checked by reading the command's own source rather than
assuming one existed because the others had one. Two ways to handle that: pretend it isn't a gap, or
say so plainly and accept the one real constraint this specific tool actually has, rather than
inventing a workaround the tool itself never offered. It shipped the second way — not because it's
the more comfortable answer, but because it's the true one.
