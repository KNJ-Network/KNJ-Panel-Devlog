# Phase 296 - Stopping Hacked Sites Sending Spam

SMTP Restrictions was a page with one tick box. Behind it is a firewall rule that stops a hacked website sending spam by
connecting straight to other servers instead of going through the server's own mail system, which is how most website spam
skips the queue, the sending limits and the signing. A good protection, on a page that did not deserve to be one, and one that
could not tell you whether it was really working.

## What changed

It is now a card on the outgoing mail tab of Mail Settings, and the menu has one page fewer. The card reads the live firewall
and says whether the protection is in place, so a switch that says On can no longer quietly be doing nothing. That check
turned up two real gaps. The rule only covered the older internet addressing, leaving the newer kind wide open, and when mail
had been handed to a separate mail server the rule was applied on that server instead of the one the websites run on. Both are
fixed.

There is now a button that tries what a hacked site would try and confirms it is refused, and a list of which hosting accounts
have tried to send directly in the last week. A site on that list is very likely compromised, so it tells you where to look.
Turning it on asks first, because anything on the server that sent mail that way will stop, and the security scan now checks it.
