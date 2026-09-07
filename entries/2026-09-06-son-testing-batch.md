# Phase 203 - A Weekend of Someone Else Actually Using It

Every bug this project has caught live so far came from one person using the panel the way it was
meant to be used and hitting something real. This batch is the first time that person wasn't the
one building it — a family member spent a weekend actually playing with the panel, doing ordinary
things an account owner would do, and found four real problems doing it.

## The button that couldn't remove anything

The Subscribers page has a "Remove selected" button for clearing out spam sign-ups in bulk — a
real need, since a public subscribe form attracts exactly the kind of junk entries nobody wants to
click through one at a time. Every click failed outright, forcing them one at a time anyway. The
cause was a one-line mismatch nobody had noticed: the button's form told the browser to send a
`DELETE` request, but the route behind it had only ever been registered to accept `POST`. The
existing test for this exact feature didn't catch it because it posts straight to the URL instead
of going through the real rendered form — proving the controller worked, never proving the button
that's supposed to reach it actually could.

## A box that only pretended to be one box

The App Installer's "Install Location" field shows the site's own domain next to a text box for
the folder name — `yourdomain.com/` followed by wherever you want WordPress installed. Both pieces
shared one visible border, which reads, at a glance, as a single field. Click on the domain part
expecting to type there, and nothing happens — it's not editable, it just looks exactly like the
part that is. The same shape existed on four fields on the Databases page too. The fix is simple
once named: the part that can't be typed into shouldn't share a box with the part that can.

## Nowhere to actually log in

After a WordPress install finishes, the page tells you it's live and links to the site itself —
but nothing pointed at the actual admin login. Guessing `wp-login.php` from memory isn't something
a non-technical account owner should have to do, so now there's a real "Log in to admin" link
right on the install-complete page and the manage-installs table, wired through a new opt-in
interface so a future app in the catalog without an equivalent login page doesn't have to fake one.

## The install that failed on a folder the panel put there itself

The real one: after creating a subdomain and trying to install WordPress into it, the install
failed with "target directory is not empty" — on a folder that had never been touched by anyone.
Every fresh domain or subdomain gets a "Website Coming Soon" placeholder page the instant it's
created, entirely so a visitor never sees someone else's site if they arrive before there's
anything real to show. Nobody had connected that placeholder to the app installer's own safety
check, which exists for a completely different, completely legitimate reason: to make sure an
install can never silently overwrite a customer's actual files. The two features had simply never
been introduced to each other, and the very first thing anyone would ever try to do with a fresh
subdomain — install something onto it — was exactly the thing that collided with both of them at
once.

Reproduced for real before writing a line of the fix: created a genuine subdomain on panel-dev and
watched the exact same "not empty" failure happen on demand, on a directory containing nothing but
the placeholder the subdomain feature had written seconds earlier.

The fix keeps the placeholder — it's still worth having — but teaches the installer to recognize
it specifically. Every place that writes the placeholder now also drops a hidden marker file
alongside it. Every install action checks for that exact marker before its emptiness check runs,
and clears the placeholder pair only when it's the *sole* thing present. A real file left behind by
an actual account owner, even one that happens to share the placeholder's own filename, still
correctly blocks the install — the marker's presence is what makes something safe to clear, not
its name or its content.

Tested and verified live on panel-dev for all four: the bulk-destroy form now actually posts, the
Install Location and Database fields now clearly show which part is editable, a completed
WordPress install now links straight to its own admin login, and — the one that mattered most — a
brand new subdomain now accepts its first WordPress install cleanly, on the very first try.
