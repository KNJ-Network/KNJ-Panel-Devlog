# Phase 304 - Who Did What

The Admin Action Log is the record of what administrators do that touches customers' data, and it was the hardest page on the
panel to read. Every entry was the same grey label, the details were a line of raw text, and a sign-in as a customer from
weeks ago still said it was active. It could not be filtered by person or date, it could not answer the question it exists
for, which is who has looked at one particular customer's data, and it recorded nothing about suspending or terminating an
account, resetting a password, managing resellers or dealing with malware.

## What changed

Each entry is now a plain sentence with a colour for the kind of thing it was: looked at customer data, signed in as a
customer, changed something, or removed something. The page can be filtered by administrator, time and kind, searched by
account, and downloaded as a CSV of exactly what is on screen. A sign-in as a customer that was never closed is called out so a
forgotten session is noticed.

A second view shows everything done to one customer, with a count of each kind, so a data request or a complaint can be
answered in a minute. The log also now records the account, reseller and malware actions it used to miss, including those made
through the API, and the page no longer fails when an API token has been created.
