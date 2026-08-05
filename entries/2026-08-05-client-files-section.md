# Phase 61 - The Client Files Section, and a Roadmap Row That Was Already True

Three items on the account-side Files roadmap, plus one that turned out to already be done. Two
bugs were found by making real HTTP requests against the server instead of relying on a green
test suite.

WebDAV access was marked Planned on the public roadmap, but the feature had already shipped as
part of an earlier batch. The roadmap row was corrected rather than rebuilding something that
already works.

Deploy from Git clones a repository into a subdirectory of a domain and redeploys on a webhook
from GitHub, GitLab, or Bitbucket. Every git operation runs as the account's own system user.
Live verification against a real repository found two bugs: a page that errored on first load
because no test had rendered it, and a webhook endpoint that needed to be exempt from form
protection. Both were fixed and confirmed with a real webhook that redeployed.

Image tools (thumbnails, resizing, and format conversion) needed no new privileged action. They
were verified with a real uploaded PNG: a thumbnail at the requested size, an in-place resize,
and a real WebP file.

That verification also surfaced an unrelated File Manager limitation at the very top level of an
account, which was tracked separately rather than folded into this phase.

Files section closed out.
