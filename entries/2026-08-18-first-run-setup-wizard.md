# Phase 98 - First-Run Setup Wizard

Spent this session doing exhaustive competitive research — cPanel & WHM's own DNS Only product,
then the full WHM/cPanel install, page by page. Deep in that, one thing kept standing out every
time it came up: cPanel's install script and first login. Worth building from directly, not just
noting for later.

## What cPanel does, and what we didn't

cPanel's installer never sets a root password itself — it assumes the box's OS-level root is
already configured, and its one-time WHM login link authenticates as that same account. First
login then walks a wizard: accept legal documents (a mandatory checkbox gate over the EULA,
Privacy Policy, and support agreement), then a trial-license activation step, landing on the
dashboard.

Our own `bootstrap-server.sh` had no legal-agreement step anywhere, and its "first admin" flow —
`knjpanel:create-admin` auto-generating a random password and printing it once at the end of the
install log — worked, but wasn't a real onboarding experience. No agreement to accept, no choice
in your own admin email or password.

## The wizard

Same one-time-link shape cPanel uses, but with a proper flow behind it. `bootstrap-server.sh` now
runs `knjpanel:generate-setup-token` instead of `knjpanel:create-admin`, printing a one-time
`/setup/{token}` link at the end of install rather than credentials. Visiting it walks:

1. **Legal agreements** — real Terms of Service and Privacy Policy content
   (`resources/legal/*.md`, rendered via `Str::markdown()`), a mandatory "I agree" checkbox.
2. **Admin account** — the operator sets their own email and password. Nothing auto-generated.
3. **Basic config** — hostname, timezone, contact email, reusing the exact same validation rules
   and provisioning-script calls `ServerSettingsController` already uses post-login.
4. **DNS-only role only** — an extra step offering to link this box to a Main server right there
   in the wizard, with an explicit "skip — link later" option. The existing post-login
   `DnsOnlySetupController` linking page stays completely untouched as that fallback.
5. **Congratulations** — a button to the login page, where the just-chosen admin credentials work.

Identical wizard for both server roles; one step differs. Gated by a completion flag
(`Setting::get('setup.completed_at')`), not "does an admin exist" — the admin gets created
partway through step 2, so gating on that would lock the operator out of the remaining steps the
moment they finished it. `EnsureSetupIsAvailable` middleware 404s on a token mismatch or once
setup is already complete, with one exception: the congratulations page itself stays reachable
afterward, since it's static and blocking it would break a post-finish page refresh.
`knjpanel:create-admin` stays in the codebase, unused by the install script now, as a manual
recovery command if a setup link is ever lost beyond regenerating a new one.

## A caught-in-testing bug

Four of the five wizard views forgot to actually pass the URL's `{token}` through to the Blade
template — every step's own form action needs it to build the next step's URL. Only surfaced once
the full feature-test suite actually rendered each view rather than following redirects past it;
the fix was threading `$this->token($request)` into every `view()` call, not just the ones whose
POST handlers already needed it.

## Verified

Full suite (1,480 tests) and `pint --test` both pass, including new coverage for: token mismatch
404s, the completed-flag blocking every step except the congratulations page, the legal-acceptance
gate blocking admin creation, password hashing and `terms_accepted_at` stamping, the DNS-only role
showing the linking step (and `main` 404ing it), and skip-linking still reaching completion. Live
end-to-end verification is the user's own next step — testing this exact flow on a brand new VM,
start to finish, is what prompted researching cPanel's flow in the first place.

## Next

Whatever the user's own fresh-install run turns up.
