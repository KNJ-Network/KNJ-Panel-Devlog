# Phase 203 - A Weekend of Someone Else Actually Using It

Until now, bugs in this project had come from the person building it using the panel the way it was
meant to be used. This batch is the first time someone else did: a second tester spent a weekend doing
ordinary things an account owner would do, and found four real problems.

## The button that couldn't remove anything

The Subscribers page has a "Remove selected" button for clearing spam sign-ups in bulk. Every click
failed, because the form sent one kind of request while the route behind it had only been registered to
accept another. The existing test didn't catch it because it posted straight to the URL instead of going
through the rendered form, proving the controller worked but not that the button could reach it.

## A box that only pretended to be one box

The App Installer's "Install Location" field shows the site's domain next to a text box for the folder
name. Both shared one visible border, which reads as a single field, so clicking the domain part and
expecting to type did nothing. The same shape existed on four fields on the Databases page. The part
that can't be typed into no longer shares a box with the part that can.

## Nowhere to actually log in

After a WordPress install finished, nothing pointed at the admin login. There is now a "Log in to
admin" link on the install-complete page and the manage-installs table, wired through an opt-in
interface so a future app without such a page doesn't have to fake one.

## The install that failed on a folder the panel put there itself

After creating a subdomain and installing WordPress into it, the install failed with "target directory
is not empty" on a folder nobody had touched. Every new domain gets a "Website Coming Soon" placeholder
so visitors never see another site, and the installer's safety check, which exists so an install can
never overwrite a customer's real files, saw it as content. The two features had never been introduced
to each other.

This was reproduced on the development server before any fix was written. The placeholder is kept, and
the installer now recognizes it specifically and clears it only when it is the sole thing present. A
real file left by an account owner still blocks the install.

Tested and verified on the development server for all four: the bulk form posts, the editable parts of
fields are clearly distinguished, installs link to their admin login, and a new subdomain accepts its
first WordPress install on the first try.
