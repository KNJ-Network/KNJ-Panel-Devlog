# Phase 140 - The PTR Check Was Looking at the Wrong Server

Same bug family as the mail-DNS fix from earlier that day, found while checking that fix live on a split
mail and web install: the Email Deliverability page's reverse-DNS (PTR) check was fixed to Main's own IP,
on both the admin and account-side versions of the page.

That is correct when mail and web share a box, but wrong once a linked Mail Only satellite is switched
on. A receiving mail server checks the sending server's reverse DNS, not the website's, so the page was
showing Main's result where mail goes out through a different server.

Both controllers now resolve the IP through the same active-mail-server lookup the earlier MX fix uses,
so the PTR check follows wherever mail is actually sent from. It is a good reminder that "the fix works"
and "everything downstream of the old assumption got fixed too" are not the same claim. Tested
(2504/2504) and live-verified.
