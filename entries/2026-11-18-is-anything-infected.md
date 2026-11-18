# Phase 303 - Is Anything Infected

The Malware Scan page showed a shield, four counters and the ten latest findings. It said "Active" next to each protection
without checking it, offered a checklist of every account to choose what to scan, and could not show findings from more than
one scan at a time. Worse, a scan where the antivirus could not run still finished as "No infected files found", which is an
all-clear the scan had not earned.

## What changed

The page now opens on a banner that changes with the state: calm and green when nothing is wrong, amber when a finding needs a
decision or a scan could not finish, red when an account is suspended for security. While scans run it turns into a live view
with a progress ring, the files checked so far, how many accounts are done, an estimate of the time left and a feed of what is
being scanned. Below it, a list says what needs a look, worst first, each with a button, and the protections are now read from
the server instead of assumed.

Findings from every account are handled in one place, there is a paged history of every scan, and every account is shown as a
square coloured by when it was last scanned, so a server with hundreds of accounts can be read at a glance and a group of them
scanned in one go.

A scan that could not fully run is now shown as incomplete, with the reason, for administrators and for the customer, and
administrators are told about it.
