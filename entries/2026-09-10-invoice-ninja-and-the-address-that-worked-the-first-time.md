# Phase 214 - The Address That Worked the First Time

The last app taught the server a new rule: give every installed app its own address, kept current
automatically, never borrowed from whatever sits at the site's own front door. The real test of a
lesson like that is whether the next app needs teaching at all.

## Nothing to fix this time

Invoice Ninja has the same shape as the previous app: no login file anywhere, every page a route decided
by code. It was installed, its login page requested, and it answered the first time with nothing written
specifically for it. The rule learned once now applies to every app that needs it.

## The install that wasn't where the obvious path led

The obvious way to install something built on the same framework as the last two apps is to clone it and
let the framework's tools finish the job. Invoice Ninja's own documentation warns against that for a real
install. The app is a framework backend behind a separate, prebuilt front-end application, and cloning
the backend alone gives half an app, with no simple way to build the rest on a typical server.

What works is a single published release file that contains both halves already built. It needs no extra
toolchain and no slow build step, and it was slower to find but faster in the end than every app
installed before it.

## The admin account

Every install has to create a real admin account without exposing the password more than necessary.
The earlier apps offered a safer way to supply it. This app's account-creation command accepts the
password only as an option, which was confirmed by reading the command's source rather than assuming.
We chose to document that as a constraint of this specific tool, rather than invent a workaround it
never offered.
