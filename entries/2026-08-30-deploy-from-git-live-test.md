# Phase 159 - Dogfooding the Deploy Button

Deploy from Git had been tested against disposable domains and framework starters. This time we used it for a real Laravel application with a frontend build step, using exactly the options the account-side UI already offers.

Clone & Link worked on the first attempt with no manual intervention. A second deploy with no new commit confirmed the deploy script is idempotent: nothing was reinstalled or re-migrated. We then registered the repository's webhook on the Git host, pushed a small change from the command line, and watched the panel pick it up and deploy it automatically. A second push reverted it the same way.

No code changed in this entry. The account-side Deploy from Git guide gained a tested worked example with the reasoning for each step, and a section confirming the webhook path against a real repository host.

Tested (2510/2510, unchanged).
