# Phase 140 - The PTR Check Was Looking at the Wrong Server

Same bug family as the mail-DNS fix from earlier today, found while checking that fix live on a
real split mail/web install: the Email Deliverability page's reverse-DNS (PTR) check was hardcoded
to Main's own IP, on both the admin and account-side versions of the page.

That's correct for the common case — mail and web on the same box — but wrong the moment a linked
Mail Only satellite is switched on. A receiving mail server checks the sending server's own reverse
DNS, not the website's. Showing Main's PTR result on an install where mail actually goes out through
a different server entirely was showing the wrong answer — confidently.

Both controllers now resolve the IP to check through the same active-mail-server lookup the earlier
MX fix uses, so the PTR check follows wherever mail is actually sent from, not always Main. Caught
live on a real four-server stack (dedicated mail, DNS, and web-facing boxes) while spot-checking the
earlier fix — a good reminder that "the fix works" and "everything downstream of the old assumption
got fixed too" aren't the same claim. Tested (2504/2504) and live-verified.
