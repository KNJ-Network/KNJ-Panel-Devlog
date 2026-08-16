# Phase 83 - Eight Packages, Verified One at a Time

The last open item in the Client Panel's own Software section: PHP PEAR Packages. The obvious
shortcut was to guess a curated set from PEAR's own well-known channel naming convention
(`Category_Subcategory` → apt package `php-category-subcategory`) — the same convention that made
last phase's Perl Modules list easy to write. It doesn't hold up nearly as cleanly for PEAR.

## The convention breaks more often than it holds

Checked every candidate directly against panel-dev rather than trusting the pattern: `php-console-
getopt`, `php-http-request2`, and `php-archive-tar` all *look* like real Ubuntu package names and
all three don't exist. `php-validate` does. The only way to know which is which is to actually ask
apt. Eight packages made it into the final curated set — Mail, Mail_Mime, Net_SMTP, Net_Socket, DB,
Auth_SASL, Validate, Log — each confirmed twice: `apt-cache policy` for the package itself, then
`dpkg -c` on the downloaded (not installed) `.deb` for its real file layout, since two of PEAR's own
packages already installed on this box (`php-mail-mime`, `php-net-smtp`, both pulled in as mail
dependencies) turned out to match the predicted `/usr/share/php/...` layout exactly — useful
confirmation the convention was right some of the time, not proof it was right every time.

## A simpler install-state check than Perl Modules needed

Perl Modules asks the interpreter directly — `perl -M<module> -e1` — because a Perl module might be
a compiled part of the interpreter itself. A PEAR package is never that; it's a plain PHP file
sitting in a shared include path. A `file_exists()` check against each package's real, confirmed
entry point is the whole job, no subprocess required at all — simpler than the Perl Modules service,
in a real way, not just a smaller diff.

## What shipped

`Account\PhpPearPackagesController` + `PhpPearPackageService`, reachable from the account panel's
own Software group — same self-service, no-admin-needed shape Perl Modules shipped with a day
earlier. That closes out the Client Panel's whole Software section.

## Next

Continuing down the gap list.
