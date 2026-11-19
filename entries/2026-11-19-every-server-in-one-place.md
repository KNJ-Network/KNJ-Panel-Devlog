# Phase 305 - Every Server In One Place

Service Status answers a simple question, is everything running, and it could only answer it for one machine. Once mail moved
to its own server the page could only say that Postfix and Dovecot were "on" somewhere else, with no status for the rest of
that server, nothing for a DNS server at all and no way to act on any of it. The three things on the screen that mattered most
in a real hosting setup were the ones it could not see.

## What changed

The page now reads every linked server as well as the main one. It opens with a summary and a short list of anything that
should be running and is not, wherever it is, with a button to start it. Each server has its own tab, and a final view lays
every service against every server so it is obvious where mail, DNS and the websites actually live.

Services are grouped by what they do and say what they are for, how long they have been up and what they are using. Restarting
one says what will pause while it comes back, and a linked server carries the restart out itself. A service's recent log can be
read from the page, with customers' email addresses hidden, and reading it is recorded. The alert for a stopped core service
now covers the linked servers too.
