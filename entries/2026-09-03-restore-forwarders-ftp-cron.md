# Phase 182 - What Else Were We Not Looking For

Finding the missing contacts raised a broader question: what else might a backup carry that the restore never looks for? We reviewed a real-world archive against what the restore reads.

Most differences were known limitations or empty for that account. Three categories stood out as having existing panel features and no restore code: email forwarders, FTP accounts and cron jobs. None needed new infrastructure, since their create and restore paths already existed for the account owner's UI and the panel's own backup.

## Judgment calls

Default boilerplate is not restored as customer data: the default catch-all rule is not a forward, and the internal statistics FTP account is not something a customer configured.

## Credentials

Each new account gets a fresh FTP login, so restoring the old one by name would leave two logins. The restore recognizes the account's primary login and applies the original credential to the existing one through the real restore path.

## Verifying

Each parser has tests built from real line formats, including a forwarder with several destinations, which this panel's record cannot hold, so the overflow is reported rather than dropped. A full restore run confirms the wiring.

Tested (2709/2709).
