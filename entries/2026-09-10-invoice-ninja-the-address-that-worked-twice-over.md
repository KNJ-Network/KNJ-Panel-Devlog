# Phase 215 - The Address That Worked, Then Didn't

The last entry ended on the first click-through working. It shouldn't have ended there.

## What the first test actually proved

A login page loading is a real signal, but it's the shape of signal that hides a smaller one right
behind it. The page loaded. Every script and stylesheet it asked for by name, up front, loaded too —
the exact list the browser reads before it does anything else, and the exact list the earlier fix
was built to correct. That's a real result. It is also, it turns out, not the same claim as "this
app works."

## The second file this app never mentioned by name

Everything that page asked for up front had its address corrected. Nothing it might ask for *later*
did — because nothing on the page, written down anywhere a text fix could reach, says what those
addresses even are. They're decided at the moment the app itself, already running in the browser,
reaches for a part of itself it hasn't loaded yet. That decision is made by code that was compiled
once, somewhere else, months before this install ever existed, and the address it reaches for is
wrong in exactly the same way the first ones were — just invisible to anything that only reads what's
written on the page.

Found the only way it could be found: not by reading, but by asking the app to actually go somewhere,
and watching what it asked for next. The same file, requested twice, at two different addresses — one
correct, one not — is about as direct as evidence gets. One rewrite reached the words on the page. The
other address was never written down anywhere words could reach.

## What doesn't get patched

A compiled file is not a template with blanks left in it. There's nothing to fill in, no seam to edit
along, nothing sitting there waiting to be told the truth. Fixing it for real means building it again,
correctly, which needs tools this install exists specifically to avoid needing at all. Patching it
convincingly is the harder path, not the easier one — length of work spent making something *look*
fixed instead of confirming it actually is.

So it comes back off the shelf. Not deleted — the parts that were right stay right, kept exactly as
they are for whenever the real fix is built. Just not offered to anyone as something that works,
because right now, past the first click, it doesn't. The list moves on to the next name on it.
