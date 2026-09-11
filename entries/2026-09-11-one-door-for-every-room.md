# Phase 232 - One Door for Every Room

A single page had been built once, for one specific mystery, months ago. Tonight's question
wasn't whether that page still worked. It was whether a page built for one mystery was still the
right shape now that the mystery was solved.

## A tool that outlived its case

The page existed because of a stubborn bug — a server returning an error with no visible cause,
every obvious script checked and cleared, no direct way to read the real failure. The page that
got built to chase it down did its job: it found the answer, the fix shipped, the bug closed.
What it left behind was a page still standing, still working, permanently associated in name with
an investigation that had already ended — a tool that kept the label of the case it was built for
long after the case itself was closed.

Asking "do we still need this" about a tool like that is the right question, but it's usually the
wrong question to answer in isolation. The honest answer wasn't yes or no. It was that the
capability underneath — reading a service's own error log without needing to shell into the
machine — was never actually about that one bug. It just never had anywhere else to live.

## The same gap, wearing different names

Looking past that one page at what else existed turned up the real shape of the problem: nothing
missing in any single spot, just no single spot covering all of it. One service had a dedicated
page built entirely around its own mystery. Several others — the web server, the database, the
process that runs the panel's own code — had nothing at all, not because they mattered less, but
because none of them had ever had their own incident dramatic enough to earn a page.

That's not really seven separate small gaps. It's one gap, worn thin across seven different
services, each with the same answer: someone will eventually want to see this log, and today
there's no consistent way to.

## Reading a log without pretending every log is the same log

Not every log lives in the same place or answers to the same question. Some are plain files sitting
on whichever machine happens to be running a particular job right now. Some are a shared history a
service manager already keeps, better asked through the mechanism built for it than duplicated
badly. And some belong to a piece of infrastructure that doesn't always live on the same machine
twice — asking the wrong box for it doesn't fail loudly, it just quietly hands back someone else's
answer, confident and wrong. Treating all of those as one uniform kind of read would have been the
easy design and the incorrect one. Keeping them as what they actually are, behind one page instead
of scattered across none, was the harder version that's actually true.

## Finding the mistake by fixing something else

Retiring the old single-purpose page into a new shared one didn't just relocate a feature — it
walked straight into a live, currently-broken path along the way, the exact error the page was
throwing that same day. The cause was mundane and had happened twice before under different names:
a capability built and tested on its own, never added to the list of things allowed to travel to
wherever it actually needed to run. Third time the same shape of gap slipped through — worth
noticing as a pattern now, not just patching as a one-off again.

## What actually finished

One page answers "show me this service's log" for every service worth asking about, instead of
one page for the service that happened to have the worst week once, and silence for everything
else.
