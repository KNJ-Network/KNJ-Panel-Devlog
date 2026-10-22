# Phase 271 - The Licence Server Grows Up

The licence server was built in a day, before most of the panel existed, and then left alone while it worked.
The panel has grown a lot since, so it was time to look at it again.

## A proper back office

The staff console used to be a list. It now opens on an overview: how many licences are paid, how many are
trials, which are waiting on a late payment, which have not been heard from lately, and which versions of the
panel are in use. Each licence has its own page with a timeline of everything that has happened to it, and
the buttons support actually needs: extend, move to a new server, add a note, revoke and restore. Customers
and a log of every change sit alongside it.

## Late payments no longer lock anyone out

A licence used to stop on the exact day it was due, so a card that needed a second attempt could lock a
customer out of their own panel for a day. Paid licences now have a few days of grace, and the Licence page
says plainly that a payment is due and when the panel would lock.

## Payments, refunds and cancellations

The server now understands failed and recovered payments, refunds and ended subscriptions, and a renewal can
never shorten a term that was extended by hand. A cancelled subscription says it ends rather than renews.

## Housekeeping

Everything the build depends on was brought up to date, the test suite that had quietly stopped working was
repaired, and the connection every panel makes to it was left exactly as it was, with the new details added
alongside.
