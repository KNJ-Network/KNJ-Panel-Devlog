# Phase 269 - Mail That Left Unsigned

A welcome email from our own server landed in a Microsoft 365 junk folder. Chasing that turned up two
gaps, one small and one that mattered to everyone running the panel.

## What happened

Every email carries a signature that proves it left from the domain it claims. The panel set this up, and
the Deliverability page reported it as fine. But the page only looked at public DNS, which shows the
published half of the signature whether or not any server could produce the other half. When we tested the
real path, nothing was being signed at all. The signing service was running and the DNS records were right,
but its setup could never match a sender, so it quietly passed everything through unsigned. No error
appeared anywhere.

The second gap was where the email left from. The panel's own mail (welcome emails, ticket notices,
password resets) was sent by the web server directly, even on a full stack with a dedicated mail server.
That mail went out from the wrong address, without the mail server's signing keys and without its tidy
reverse DNS.

## What changed

- Signing now works. New mail servers are set up correctly, and existing installs are repaired
  automatically on their next upgrade. No key is replaced, so published DNS records stay valid.
- When a mail server is linked, the web server hands its own mail to it. The mail server trusts only that
  one address, the arrangement is checked before it is used, and if anything fails, mail keeps leaving as it
  did before. One-server installs behave exactly as they did.
- The Deliverability page has a new card that asks the sending server itself whether it can sign, shows
  where the panel's mail really leaves from and what that address's reverse DNS says, and offers a repair.
- Subscriber emails now include a plain-text version and the one-click unsubscribe that Gmail and Outlook
  look for on newsletters.

The lesson: a check that reads the published record is not a check that the thing works. Ask the machine
that has to do the job.
