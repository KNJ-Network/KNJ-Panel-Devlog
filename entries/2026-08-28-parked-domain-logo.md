# Phase 136 - The Placeholder Logo Nobody Had Replaced

A small one: the static "nothing here yet" page nginx serves for any domain pointed at a server
with no matching hosting account was still showing a generic blue "K" letter badge instead of the
real KNJ mark — noticed while checking a freshly pointed domain on a production-style install.

Swapped it for the same logo used everywhere else in the panel (sidebar, login page), inlined as a
base64 data URI since this particular page is plain static HTML with no Laravel asset pipeline
behind it — it's copied once to `/var/www/panel-landing/index.html` at install time and served
directly by nginx, entirely outside the app's own request cycle.

That "copied once, outside the app" detail is also what made this worth a second look: the dev
server's own autodeploy path already knew how to resync that file on every deploy, but the real
customer-facing upgrade script never did — so a fix here would have shipped in a release and then
silently never reached an already-installed server. Added the same resync step there too, confirmed
against the dev server: the resynced file matches the source byte-for-byte.
