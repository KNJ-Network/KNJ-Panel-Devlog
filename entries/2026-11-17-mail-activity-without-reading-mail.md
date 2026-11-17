# Phase 292 - Looking After Mail Without Reading It

The Mail Queue page listed every waiting message with its sender and every recipient, and the delivery reports and statistics
pages did the same for the mail log. That is fine on a server you run for yourself. On a server you sell hosting from, the
people writing those addresses are your customers' customers, their addresses are their own personal data, and the person
running the server is not entitled to browse them just because the page happens to show them.

## What changed

The three pages are now one, Mail Activity, with a tab each for the queue, the history and the statistics. It is organised
so that you do not need an address to do the job. It opens with counts, the reasons mail is waiting in plain groups, and the
domains it is waiting for, and anything you might want to do to mail you can do to a whole domain's worth or to one message
by its reference, without seeing who it is for.

Addresses are shortened everywhere, including where another server's reply quotes one and in the recent deliveries on the
mail-only server's dashboard. To see them in full you choose a reason and they are shown for fifteen minutes, then hidden
again. Every time addresses are shown, an address is searched for or queued mail is deleted, it is recorded with who, when and
why, and the latest are listed on the page itself. Subjects and message text are never read at all.

You can now hold a domain's waiting mail while you find the cause, and release it afterwards. Deleting needs a reason, says
exactly how many messages it will remove, and refuses to go ahead if the queue has changed since you looked. The button that
deleted the whole queue in one click has gone.
