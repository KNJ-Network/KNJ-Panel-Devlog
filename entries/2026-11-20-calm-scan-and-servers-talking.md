# Phase 315 - A Calm Scan, And Servers That Talk Again

The first look at the new release on the real servers found one fault and one thing that felt wrong. After the upgrade, the
page that checks every service on every server said the name servers and the mail server could not be reached, when all
three were fine: the call between servers was being refused. And at the start of a malware scan the page showed a
frightening speed and a time left of hours, because it averaged from the moment the scan was queued, including the time
spent counting files.

## What changed

The servers can ask each other for their service status again, and a test now checks every call one server makes to
another so this cannot go unnoticed a second time. The scan page shows the real recent speed, says nothing about time left
until it can judge honestly, shows how long the scan has been running, and counts up smoothly. The same release brought
thirteen out-of-date components up to date and added limits to calls between servers and to a button a customer could press
over and over.
