# Phase 159 - Dogfooding the Deploy Button

Deploy from Git has been live since Phase 439 in the private roadmap, tested against disposable
domains and a handful of framework starters. It had never been used for anything that actually
mattered. Today it deployed the KNJ Network marketing site itself — the real Laravel app behind
knj.network, from its own private GitHub repository, replacing nothing but an empty account with
no prior content.

The setup was exactly what the account-side UI already offers, nothing added for the occasion: an
SSH deploy key generated on the account, added as a read-only key on the repository, an SSH-style
`git@github.com:...` URL, Public Directory set to `public` so the Laravel checkout's own files
(`.env`, `app/`, `.git/`) stay outside the web root, and a Deploy Script covering the real sequence
a Laravel app with a Vite build actually needs — `.env` bootstrapping from `.env.example` (the repo
defaults to SQLite, so a `touch database/database.sqlite` earns its place too), `composer install`,
`npm ci && npm run build`, `migrate --force`, and the three `artisan ...:cache` calls at the end.
Clicked Clone & Link once. It worked on the first attempt — no retry, no manual SSH intervention,
no hand-editing anything on the server afterward.

A `Deploy Now` immediately after (simulating a routine redeploy with no new commit) proved the
script is safely idempotent: `cp -n` left the already-configured `.env` alone, composer reported
nothing new to install, and `migrate --force` reported nothing to migrate. That's not a given for
hand-written deploy scripts in general — it's a given here because the guide (now updated with this
exact worked example) tells people to write it that way, and it held up against a real one.

Then the other half — the part that doesn't get exercised just by clicking a button in the panel:
registered the panel's per-repository webhook URL directly on GitHub, pushed a one-line visible
change straight from the command line with no visit to the panel at all, and watched the linked
repository's card pick it up on its own — `Running`, then `Completed`, new commit hash, change live.
Reverted with a second push; same automatic pickup, site back to normal. Two real pushes, two real
automatic deploys, zero clicks in the panel for either one.

Nothing shipped in this entry — no code changed. What changed is the account-side Deploy from Git
guide, which now carries a real, tested Laravel-plus-Vite worked example (with the reasoning for
every line of the deploy script, not just the script itself) instead of only the generic
walkthrough it had before, and a short section confirming the webhook path was proven against
a real repository host, not just described.

Tested (2510/2510, unchanged — no code touched).
