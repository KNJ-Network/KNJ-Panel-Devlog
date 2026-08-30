# Phase 164 - The Account That Wouldn't Leave

A pre-launch readiness audit of the real 4-server production stack (Main, Mail Only, and two DNS-only
satellites) — the last checkpoint before treating the install as ready for real customers — created a
disposable test account with one mailbox specifically to prove the whole pipeline end to end: DNS zone
sync to both nameserver satellites, and real mail delivery through the linked Mail Only server. Both
worked. Then came the part that's supposed to be routine: delete the test account and confirm nothing
is left behind.

DNS was clean everywhere — `dig SOA` came back NXDOMAIN on Main and both `ns1`/`ns2` satellites within
seconds of clicking Remove. The Mail Server told a different story. SSHing in and grepping
`/etc/postfix/virtual-mailbox-domains` and `virtual-mailbox-maps` turned up the deleted domain and
mailbox, still sitting there, still valid Postfix config, still routable.

## Why DNS self-healed and mail didn't

`DeprovisionAccountJob` deletes an account's mailboxes the same way it deletes almost everything else
about that account: `$account->owner?->delete()` cascades the `accounts.user_id` FK, and `sites`/
`mailboxes` both cascade off that at the database level. Fast, simple, and correct for the DB itself —
the row really is gone. But a raw FK cascade fires no Eloquent model event, and every single place in
this codebase that keeps a linked Mail Only satellite's Postfix/Dovecot config in sync with the
database — `MailboxService::createMailbox()`/`deleteMailbox()`/`changePassword()`/`updateQuota()`/
`suspend()`/`unsuspend()`, all four of them — does that syncing by explicitly calling
`MailAuthMapService::regenerate()` right after the row change, inside the same transaction. Account
deletion never called it. Not a race, not a timing issue — the call just isn't there, on this one path
and this one path only, because account deletion is the one place mail rows disappear without going
through `MailboxService` at all.

Went looking for a safety net before concluding this was a real gap: DNS-only zone membership has
`SyncDnsOnlyServers`, a `everyFifteenMinutes()` reconciliation sweep that would have caught exactly
this kind of drift on its own within 15 minutes even if the original push had failed. Mail has no
equivalent — grepping the whole scheduler in `bootstrap/app.php` and every caller of
`authMap->regenerate()` across the app turns up nothing that runs on a timer. Whatever
`DeprovisionAccountJob` leaves behind in a satellite's auth maps stays there forever, correct only by
accident whenever the next unrelated mailbox change happens to trigger a full rebuild.

## The fix, and why it has to be four calls, not one

`MailAuthMapService` only covers plain mailboxes. Catchalls, mailing lists, and forwarders are Postfix
`virtual_alias_maps` too, but they're handled by three sibling services —
`DefaultAddressAuthMapService`, `MailingListAuthMapService`, `ForwarderAuthMapService` — because none
of them share a script action or a temp file with mailbox routing. All four get the exact same
treatment: injected into `DeprovisionAccountJob::handle()`, called once each after the account row is
actually gone, wrapped the same best-effort way the job already treats database drops and addon-domain
cleanup — a failed push to an unreachable satellite must never leave an account stuck undeletable, so
each call gets its own try/catch and a logged error rather than a thrown exception. Every one of the
four `regenerate()` methods already no-ops when no Mail Only server is linked, so this runs safely on
every install regardless of role — Main-only, Mail Only itself, doesn't matter.

This test case only ever exercised the plain-mailbox gap (it's the only mail construct the disposable
account had), but the identical bug shape exists for the other three by construction, not by
inspection — same cascade-delete-with-no-event pattern, same missing call. Fixing all four in one pass
beat fixing the one that happened to get caught live and leaving the rest to surface the same way,
later, on a real customer's account.

Two new regression tests: one confirms a linked Mail Only server actually receives a fresh,
mailbox-free `mail-virtual-maps-write`/`mail-dovecot-authmap-write` push after deletion; one confirms a
failed push still lets the account finish deleting rather than getting stuck in `Deleting` forever.
Tested (2563/2563).

## Cleanup, the instruction that came mid-audit

Partway through this same audit session, a direct reminder landed: the 4-server stack under test is
the real production stack, not a sandbox, and anything created on it for testing has to be removed
afterward — no exceptions, no leftovers. That's the instruction this whole bug traces back to: the
manual post-deletion check this readiness audit did (SSHing into the Mail Server and grepping its auth
maps by hand, rather than trusting the panel's own "account removed" confirmation) is exactly the kind
of verification a routine deletion shouldn't need, and exactly the kind of check that caught this.
Once the code fix shipped, the residual auth-map entries for the disposable test domain were themselves
cleaned up by the fix doing its job on the next mailbox change — no manual file editing on a live
server, in keeping with this project's own standing rule against exactly that.

Released as v0.17.15.
