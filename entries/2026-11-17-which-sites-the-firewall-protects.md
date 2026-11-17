# Phase 298 - Which Sites The Firewall Protects

The web application firewall page showed an engine status, one switch, a strictness setting called "paranoia level" with no
explanation of what the four levels do, and a list of blocked requests identified only by rule number. It could not tell you which
of the sites on the server were actually protected, and it reported itself as degraded while it was in fact blocking attacks (that
was a fault in its own test, fixed earlier).

## What changed

The page now answers the three questions an administrator has. Is it working: a strip says so, with the reason in red if not.
Which sites are protected: every site is listed with whether it is blocking, only watching or off, and its mode can be changed
from the list. What is it stopping: attacks are named in words, such as database commands hidden in a web address, or a request
for a file that must never be public, with what each means.

The four strictness levels are explained one by one, with how likely each is to block a real visitor and a warning on the strict
ones. The list of blocked requests shows only the page that was asked for, never what followed the question mark or what was
typed into a form, because that can contain a visitor's personal details, and the firewall's own test attack no longer appears in
it. Changing a customer's site protection from here is written to the record of administrator actions.
