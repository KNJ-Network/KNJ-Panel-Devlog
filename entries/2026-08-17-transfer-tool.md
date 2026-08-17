# Phase 92 - Transfer Tool

The last missing piece of the original migration-tools roadmap line: pulling one or more accounts
straight from a live remote cPanel/WHM server over SSH, rather than requiring someone to generate a
backup archive by hand and upload it through Restore a cPanel Account first. This is what WHM calls
its own Transfer Tool.

## Reuse, not reinvention

This feature's job ends once a valid archive is sitting on this server. Everything after that,
validating the archive, detecting the domain, creating the account, and restoring files and
databases, is the same import pipeline Restore a cPanel Account already uses, unchanged. The only
change to the import service is a small additive refactor so a locally pulled file can feed the
same path an uploaded one does. On the remote end it uses the same packaging tool WHM's own
transfer feature uses, rather than reimplementing account packaging.

## Key-based authentication only

There's no password option: key-based authentication is what cPanel's own documentation steers
admins toward, and skipping passwords avoids handling another sensitive credential. Remote commands
run through `sudo` rather than assuming a root login. That was a design correction mid-build, since
the first pass authenticated as root, which is the wrong default for a security-conscious server.

## Two bugs only a live test would catch

Both were found running an actual pull against a real remote machine, and neither was reachable
from the automated tests.

**A template escape collision.** A source's `user@host:port` display rendered as literal template
syntax because the adjacent expressions formed a pattern the template compiler treats as an escape.
It was fixed by building one interpolated string.

**Line endings in pasted keys.** Browsers submit text areas with CRLF line endings, which OpenSSH's
key parser rejects. Line endings are now normalized when the key is used, so a key stored earlier
corrects itself without re-adding the source.

## Live-verified end to end

A synthetic but format-faithful packaging tool on an independently controlled machine produced a
real archive with a website file and a database containing a known marker row. A full pull through
the browser UI (connect, list accounts, transfer, import) landed both on the target server
byte-for-byte, owned by the new account's own system user.
