# Phase 105 - Mail Only, Stage 2: The Dispatcher, and a Real Selector

## The piece DNS-only never needed

DNS-only's whole design rests on one fact: zone content is always authored on Main, so the
satellite only ever receives a push of something that already exists. Mail doesn't get that
luxury. Creating a mailbox, changing a password, applying server-wide Postfix limits — these are
all things an admin triggers *inside a single request* and expects a real result back from
immediately, not an eventually-consistent background sync. Building genuine mail provisioning
meant building the piece DNS-only's design never had to: a way to redirect a privileged action to
a different box and get its result back synchronously, in the same request.

## One interception point, not twenty

Every mail-touching service in this codebase shells out to the same privileged provisioning
script the same way: `Process::run(['sudo', SCRIPT, $action, ...$args])`. That shape repeats
across roughly twenty service classes. Rather than teach each of them individually about a linked
Mail Only server, `ProvisioningScriptRunner` becomes the one place that decision gets made —
local, unless a Mail Only server is actively selected (`Setting::get('mail.active_server_id')`),
in which case it's dispatched instead. Every existing caller keeps calling the same script action
by name with the same arguments; only where it actually runs changes, and only for installs that
have deliberately opted into a linked mail server. Every existing install, unchanged, sees zero
behavior difference — confirmed directly: `MailboxService`'s and `MailServerSettingsService`'s
existing local-path test coverage (`MailTest`, `MailSettingsTest`, plus `GreylistingTest`/
`MailFilterTest`, which share the same settings-apply path) passed completely unchanged after the
migration, no test edits needed.

The remote side reuses DNS-only's own established shape rather than inventing a new one: a
synchronous authenticated POST to `/internal/mail-dispatch/{token}` (same unguessable-secret-in-
path pattern as every other `/internal/*` endpoint here), with the same hostname-then-IP-fallback
behavior `PushZoneMembershipJob` already uses for exactly the same "a freshly-linked box's
hostname might not resolve yet" reason. `MailDispatchController`, the receiving side on a Mail
Only box, needs zero mail-specific business logic of its own — it just calls the identical
`ProvisioningScriptRunner::run()` locally, which naturally takes the local branch there since a
satellite never has its own `mail.active_server_id` set.

One deliberate generalization: the temp-file-secret handling every password-touching action used
to duplicate for itself (`MailboxService::writeTempPassword()`, three separate call sites) now
lives once, inside the runner — write to a tightly-permissioned temp file, pass the path, delete
it after, whichever box actually runs the script. It also turned out to be exactly the right shape
for non-secret file content too: `MailServerSettingsService`'s generated Postfix config, migrated
the same way in the same pass, needed no new mechanism of its own.

## Proof case: mailbox create/delete, and one real feature besides

`MailboxService` — create, delete, change password — is the proof case the staged plan called for:
the first genuinely remote-capable mail action, migrated end to end and covered by a new
`ProvisioningScriptRunnerTest` exercising both the local and remote branches (including the IP
fallback and a defensive check that a stale/invalid `active_server_id` fails safe to local rather
than throwing).

Past just the proof case, this pass also shipped the actual user-facing switch the whole feature
exists for: a "Where does mail run?" selector on Mail Settings — Local (the default, unchanged
behavior) or any linked Mail Only server, restricted to servers that have actually completed the
two-key handshake (`Server::hasCompletedHandshake()`), not just been added. Switching re-applies
the current settings immediately so the newly active target picks them up without a second save.
The page is explicit that existing mailboxes aren't copied over automatically yet — that's a
separate bulk-resync job, not implied by the selector working.

## Still ahead

The remaining ~20 mail services (DKIM generation, mail queue, delivery logs, statistics, and the
rest) still call the provisioning script directly rather than through the dispatcher — Stage 3.
The `mail-catchall-ensure`/`mail-mailinglist-ensure` local-MySQL-query problem identified during
planning is untouched. So is the Service Status "Main Server"/"Mail Server" split, the bulk
existing-mailbox resync job, and cross-server admin SSO (Stage 4). Genuinely live-testing the
mailbox-create round trip against a real linked Mail Only box also hasn't happened yet — that
needs a real second server, which is in progress separately.

## Verified

Full local suite (1593 tests) and `pint --test` both green throughout. No live VM test this pass —
code-complete and unit/feature-tested via `Process::fake`/`Http::fake`, pending a real linked Mail
Only box to prove the round trip against, same bar as Stage 1's own live-verification gap.
Committed and pushed as `fc78c61` (dispatcher + mailbox proof) and `3a2d7de` (selector +
settings-apply migration).
