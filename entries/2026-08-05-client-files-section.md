# Phase 61 - The Client Files Section, and a Roadmap Row That Was Already True

Three items on the account-side Files roadmap, plus one that turned out to already be done. Two
real bugs found the same way as every other phase this session: by insisting on a real HTTP
request against the real server instead of trusting a green test suite.

WebDAV access was marked Planned on the public roadmap, but the actual feature — a full-access
login created automatically for every account, plus additional scoped logins jailed to a
subdirectory — had already shipped days earlier as part of the Service Configuration batch. The
roadmap row was just never flipped. Fixed the record rather than rebuilding something that already
works.

Deploy from Git clones a repository into a subdirectory of a domain and redeploys on a webhook —
GitHub, GitLab, or Bitbucket, since none of their payload shapes are actually parsed; every trigger
just redeploys the branch the repo was configured with, authenticated by the unguessable token in
the URL. Every git operation runs as the account's own system user, with `GIT_ALLOW_PROTOCOL=https`
set explicitly as the real transport boundary rather than trusting the URL's own scheme string.
Live-verifying it against a real GitHub repository found two bugs in one pass: a `Collection::load()`
call on a `flatMap()` result that doesn't have that method (500 on the very first page load — no
test had ever actually rendered the page, since every existing test only POSTed to an action), and
a missing CSRF exemption for the webhook route, since it's the first public POST endpoint this app
has ever had — Laravel's own test client bypasses CSRF entirely regardless of route configuration,
so nothing in the suite could have caught it. Both fixed and confirmed with a real webhook POST
that actually redeployed.

Image tools — thumbnail generation, in-place resizing, and format conversion — turned out to need
no new provisioning script action at all. `public_html` is already group-writable by the panel
process for exactly the reason Quick Site Publisher established last phase, so this is PHP's GD
extension working entirely in-process, reusing File Manager's own path-resolution boundary rather
than duplicating it. Live-verified with a real uploaded PNG: a real thumbnail at the requested
dimensions, a real in-place resize, and a real WebP file confirmed with `file`, not just a renamed
extension.

That live-verification also surfaced a bug outside this phase's own scope: uploading a file at the
very top level of File Manager, before navigating into any folder, has apparently never worked —
the account's home directory itself is locked down tighter than `public_html` underneath it, so the
panel process can read it but can't write new files directly into it. Flagged as its own follow-up
rather than folded into this phase, since fixing it properly means deciding whether to loosen a
security-relevant permission account-wide or restrict the UI instead — a real decision, not a
one-line fix.

Files section closed out. Four items, two real bugs, one already-shipped feature the roadmap hadn't
caught up to yet.
