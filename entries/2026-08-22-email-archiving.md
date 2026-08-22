# Phase 126 - Email archiving

The ninth item off the "genuinely zero" list. The archiving feature itself was the easy part; the
story here is two mail-routing bugs caught during live testing on the dev server.

## The design: a copy, not a real mailbox

Every message (incoming, outgoing, or both, independently toggleable) is silently copied to an
internal-only archive address. It deliberately is not a real mailbox record: mailboxes are tied to
a site and account, so a clean archive mailbox would have meant either a fake account or new
nullable schema. Instead the archive lives outside the app's data models, as static configuration
layered on top of the real mail setup, the same approach the panel uses elsewhere. Retention is
configurable in days (0 = forever) and pruned by a daily scheduled command.

## Bug #1, caught on the dev server: a lookup type needed a package that was not installed

The first routing implementation used a map type that requires a separate package not present on
the dev server. Once archiving was enabled, mail on that dev server failed to deliver. No customer
system was involved. We restored delivery on the dev server first, confirmed with a real SMTP test,
and then fixed the cause by switching to a built-in map type that needs no extra package.

## Bug #2, caught live: accepting a message is not the same as delivering it

Even after that fix, archive copies were still lost while normal delivery worked. The mail transfer
agent's "will I accept mail for this address" lookup is separate from the delivery agent's own user
database, which had no knowledge of the archive address, so the copy was bounced. Fixed the same
way the transfer-agent side was: a static fallback user database consulted only when the main one
finds nothing, validated like every other configuration-touching action here (check, backup, revert
on failure, reload on success).

## Verified

Live on the dev server, not just through the 11 new tests (which cannot exercise real mail
binaries): a real message was delivered to its recipient and archived with intact headers, the
archive list and download actions parsed and returned it byte-identically, and disabling archiving
cleared the configuration with the mail server's config check staying green. Full suite 1,795 and
code-style check green. Both fixes landed in v0.16.64 after the initial ship in v0.16.63.
