# Phase 170 - The Password Nobody Should Be Able to Reuse

Reported straight from the operator's own admin workflow: the Reset Password page for a customer
account could auto-generate or manually set a new password, and that was it — shown once in a
session-flash banner, never emailed anywhere. Two things were missing: an option to actually email
the new password to the account owner, and something cPanel/WHM doesn't really offer at all —
a real "send a temporary password" mode, safe enough to trust with an account takeover if it's built
wrong. The operator was explicit going in: plan this one properly, this is a security feature, not a
UX tweak.

## Ruling out the wrong foundation first

Before designing anything new, the ask was to check whether the panel's existing "Forgot password?"
link already did something like this. It doesn't, but it's also not dead code — an earlier guess this
session had wrongly assumed it was broken. It's Fortify's own `resetPasswords()` feature, auto-
registering its own rate-limited routes outside `routes/web.php` entirely. Confirmed live via
`php artisan route:list`, left untouched — it's a different flow (click a link, set a password
directly) from what this feature needed (log in *with* a temporary password, then get forced to set a
real one), and conflating the two would have been a mistake.

## Three tracking columns, and none of them mass-assignable

`password_is_temporary`, `password_temp_expires_at`, `password_temp_used_at` — new columns on
`users`, deliberately left out of `User`'s `#[Fillable(...)]` allowlist. Every write goes through
`forceFill()->save()`, matching the pattern Fortify's own password-reset action already uses
elsewhere in this codebase. The point isn't style — it's that these three columns can never be
touched by accident from some unrelated `$user->update($request->all())`-shaped call down the line.

`AccountProvisioningService::resetPassword()` grew a third `passwordMode`, `'temporary'`, alongside
`auto`/`manual`. Every call — regardless of mode — unconditionally resets all three tracking columns,
so running a normal auto/manual reset against an account with a still-pending temporary password
correctly clears that pending state too, not just the password value itself.

## Where the check belongs: after the hash, not before

The natural instinct is a new pre-login pipeline stage, the same shape as the 2FA/security-question
challenge screens. Wrong model — those fire *before* `Auth::login()` ever runs. This needed to happen
inside `Fortify::authenticateUsing()`, the single closure that already does the credential check plus
the suspended/deleting/reseller-suspended rejections. Added two more checks there, both firing only
*after* `Hash::check()` has already succeeded:

```php
if ($user->password_is_temporary) {
    if ($user->password_temp_used_at !== null) {
        throw ValidationException::withMessages([...'already been used...']);
    }
    if ($user->password_temp_expires_at?->isPast()) {
        throw ValidationException::withMessages([...'has expired...']);
    }
}
```

That ordering matters: a wrong-password guess against a temp-password account gets the same generic
failure message as any other wrong guess. Nothing about a bad guess ever hints that the account has a
pending temporary password waiting to be used, expired or not.

## The guard-name trap

Marking a temporary password "used" happens in `AppServiceProvider`'s existing `Login` event
listener — one line added to the closure already there for Security Questions' known-IP recording.
The trap: this codebase has two guards on one `users` table. `'web'` is a real login. `'account'` is
populated *only* by WHM impersonation (`AccountController::impersonate()`), a password-less shortcut
that never touches `authenticateUsing()` at all — but it still fires a `Login` event. Without an
explicit `$event->guard === 'web'` check, an admin clicking "Log in as user" on an account with a
pending temporary password would have silently burned it before the real owner ever got a chance to
use it themselves. Wrote a dedicated test for exactly this — impersonate, then assert the temp
password is still unused — since it's the one edge case in this whole feature that's easy to get
wrong and easy to not notice was wrong.

## The forced gate

Modeled directly on `EnsureLicenceIsValid`, the existing pattern for "you must do X before reaching
anything else": a new `EnsureNewPasswordIsSet` middleware, added to the same big authenticated route
group that wraps the entire Controller and Account areas, redirecting to `/password/set-new` whenever
`password_is_temporary` is still true. That page itself sits in its own small carve-out group —
reachable while gated, still behind `licence.valid` since a temporary-password login shouldn't bypass
a lapsed licence lock. No old-password re-entry required to set the new one — reaching an
authenticated session with the temporary password already proved possession of it, the same trust
Fortify's own token-based reset extends.

## Two notifications, one already-established pattern

Both follow the codebase's one convention for account-related email — `Illuminate\Notifications\
Notification` + `toMail()`, dispatched via `Notification::route('mail', $email)->notify(...)`
on-demand routing, not `$user->notify()`. `AccountPasswordChanged` for the email-the-password
checkbox on auto/manual modes; `AccountTemporaryPasswordIssued` for the new mode, spelling out the
one-time-use/24-hour/forced-change terms explicitly so nobody's surprised by any of it.

On the UI side: temporary mode always emails and never flashes the password to screen — the email is
the only disclosure channel for that mode, by design. The email checkbox is disabled outright (not
just hidden) when temporary mode is selected, so it can never submit alongside it and create an
ambiguous double-send.

## Verifying it for real, not just in tests

2610/2610 passing locally, including a new admin-impersonation-guard test and a full run through
`SetNewPasswordController`'s redirect logic from every gated area. Then panel-dev, end to end, no
shortcuts: triggered a real temporary password through the actual admin UI, read the real delivered
email off the Maildir file over SSH (Postfix → Dovecot LMTP, `MAIL_MAILER=sendmail` on panel-dev),
logged in as the account owner with it, confirmed the forced redirect fires immediately and blocks
direct navigation everywhere else, confirmed `PasswordPolicyService` rejects a weak replacement and
accepts a real one, confirmed all three tracking columns clear on success, confirmed the spent
temporary password is rejected on a second attempt afterward, confirmed the account's real dashboard
is reachable normally once a real password is set. Also re-verified the email-the-password checkbox
path separately for auto-generate mode — flash banner still appears (unlike temporary mode) and a
real `AccountPasswordChanged` email arrived with the right content.

Small unrelated fix folded into the same pass: Manage Servers' "Last Seen" column showed "Never" for
the Main server itself, since `last_seen_at` is only ever set by health-checking a *linked* server —
the Main server never gets checked against itself, so it always fell through to a fallback that reads
as broken when it's simply not applicable. Now shows `—` specifically for the self row, `Never` stays
the fallback for a genuinely never-checked linked server.

Tested (2610/2610).
