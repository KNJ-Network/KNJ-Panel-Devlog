# Phase 83 - Eight Packages, Verified One at a Time

The last open item in the Client Panel's Software section was PHP PEAR Packages. The obvious
shortcut was to guess a curated set from PEAR's channel naming convention mapped onto Ubuntu
package names, the same convention that made the Perl Modules list easy to write. It doesn't hold
up nearly as cleanly for PEAR.

## The convention breaks more often than it holds

Every candidate was checked directly against the development server rather than trusting the
pattern. Several names that look like real package names don't exist, while others do, and the only
way to know which is which is to ask the package manager. Eight packages made it into the final
curated set (Mail, Mail_Mime, Net_SMTP, Net_Socket, DB, Auth_SASL, Validate, Log), each confirmed
twice: by package metadata, then by inspecting the file layout of the downloaded package.

## A simpler install-state check than Perl Modules needed

Perl Modules asks the interpreter directly because a module might be compiled into it. A PEAR
package is never that: it's a plain PHP file in a shared include path, so a file-exists check
against each package's confirmed entry point is the whole job, with no subprocess required.

## What shipped

A new account-side page in the Software group, with the same self-service, no-admin-needed shape
Perl Modules shipped with a day earlier. That closes out the Client Panel's Software section.

## A latent bug under every package-based feature

Live-testing the Install button surfaced something bigger than a PEAR-specific bug: a "Read-only
file system" error when installing a package the server had genuinely never had before. The web
application's service hardening blocked writes to locations a brand-new package needs. Other
package-based features shared the exposure but had never hit it, because every package they were
tested against had already been installed by something else. A manual SSH test never reproduces it,
since only a real click through the UI runs inside the restricted environment. It was fixed and
re-verified live with a package the server had never touched, installed end to end through the real
button.
