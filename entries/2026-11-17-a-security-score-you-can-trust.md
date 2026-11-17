# Phase 294 - A Security Score You Can Trust

The Security Scan page ran six checks, only when someone pressed a button, and kept nothing. The result on the screen could be
weeks old and still say "100, Healthy". It also looked at none of the things the panel already knows matter: whether
administrators have two-step sign-in, whether backups are working and kept somewhere else, whether the certificate is about to
run out, whether a site is on a version of PHP that no longer gets security fixes, whether mail is being checked at all.

## What changed

It now runs nineteen checks in five areas, and every one says what it found, why it matters and where to fix it, with a button
for the three things that are safe to fix in one click. The page opens with the score and a short list of what to do first, most
serious first, and lists everything that passed underneath so a clean result is something you can read.

The server scans itself every week by default, or every night, and says plainly when a result is old. A history shows whether
things are getting better, and an alert goes out once when something turns critical or the score drops sharply.

Not every warning is a mistake. A port opened on purpose can be accepted with a reason and a date to look again. It stops
counting against the score, stays visible with the reason, and the decision is written to the log that records who did what.
A check that cannot be read is shown as a warning and never as a pass.
