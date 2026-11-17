# Phase 295 - Who Can Sign In, In One Place

The Security Policies page was mostly a summary of other pages, with one real control on it: making two-step sign-in
compulsory for administrators and resellers. That control was all or nothing. Turn it on and anyone who had not set it up was
sent to do so at their very next click, which is a good way to lock a colleague out in the middle of a task. The password rules
and the security question each lived on a page of their own, a single form and a single switch.

## What changed

Those three are now tabs of one page: passwords, two-step sign-in, a security question for unfamiliar places, and how long an
idle session lasts. The firewall and the other network controls stay where they are, shown on the overview as status.

Requiring two-step sign-in now comes with a grace period you choose. Everyone who has not set it up is emailed, sees a
reminder with the days left on every page, and carries on working, and only after that are they taken to set it up. The page
says exactly who would be affected before you save, lists everyone with whether they have it, and can remind one person.

The password rules appear as a sentence that updates as you change them, with a box to try a password against them, checked in
your browser and sent nowhere. There is a new setting to sign out idle administrators sooner than the default two hours; a page
that only refreshes itself in the background does not count as someone being there.
