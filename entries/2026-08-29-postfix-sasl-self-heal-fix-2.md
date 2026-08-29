# Phase 155 - The Self-Heal That Healed Nothing

v0.17.6 shipped to fix a confirmed bug: the mail transfer agent on an already-provisioned Mail Only
satellite could not reach the mailbox server's authentication service, because the stock configuration
ships that connection switched off and nothing had gone back to enable it on that box. The fix made the
check part of what runs whenever a server's hostname or certificate is confirmed.

After rolling v0.17.6 out to the test servers, re-confirming the affected box's unchanged hostname ran
the job cleanly in seconds, but the configuration afterward was still switched off.

The fix's own guard condition was the bug. It decided whether the listener already existed by searching
the file for the listener's path, and the stock file's commented-out stub contains exactly that string.
The search ignored the leading comment marker and matched the dead stub as happily as a live listener, so
the self-heal never ran. The same flaw was in the mail setup script's own check.

Both now require a match against an active listener line, anchored at the start of a line, so a comment
prefix can never satisfy it. We verified the corrected pattern against both the stock commented text and
a properly patched block before relying on it.

The lesson is not new: a "does this string exist in the file" check is not a safe proxy for "is this
feature active" when the stock template already contains the string as inert filler.

Tested (2506/2506).
