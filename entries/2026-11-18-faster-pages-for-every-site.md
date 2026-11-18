# Phase 299 - Faster Pages For Every Site

Every website the panel hosts, and the panel's own pages, was being served the oldest way a web server can: a visitor's
browser opened a handful of connections and queued every file of a page down them one after another. A page with a lot of
images and scripts therefore loaded slower than it needed to. It was found while checking this website's own search and
speed results, where the same page showed the old protocol.

## What changed

Every secure connection on the server now uses the faster protocol, HTTP/2, which fetches a page's files side by side.
New sites are set up with it, and existing sites are brought up to date by the next upgrade and by the panel's regular
check, including the ones where the certificate tool wrote the connection settings itself. The change is tested against
the web server's own configuration check before it takes effect, and anything the web server refuses is put back exactly
as it was, so a site is never left worse off. There is nothing to switch on.

Browsers are also now told how long they may keep a file. Pictures, audio, video, fonts, stylesheets and scripts are kept
by the visitor's browser instead of being downloaded again on every page and every visit. The default keeps pictures for a
week, fonts for a month and stylesheets and scripts for a day, the shortest time going to the files people edit and then
expect to see change. It can be lengthened or switched off from the Nginx Settings page. It only ever applies to files the
web server hands over directly: nothing a site's own code generates is given a lifetime.
