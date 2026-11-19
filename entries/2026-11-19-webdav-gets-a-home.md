# Phase 309 - WebDAV Gets A Home

WebDAV was a whole page for a single tick box, and looking behind that box found it did less than it claimed. Switching it off
only stopped new logins being created: every existing login carried on working, and nothing said so. Nothing blocked an
address that kept guessing WebDAV passwords, although FTP, mail and the panel's own sign-in all do. Every login could change
and delete files, with no read-only choice, and a hidden size limit sat in the generated configuration where nobody could see
it. The administrator could not see who had a WebDAV login at all.

## What changed

WebDAV moved onto the FTP page, which became FTP and WebDAV, since both are the same job: how customers get files onto the
server. The page now answers for both whether it is safe and working, who has been trying to use it, and what needs attention.

Switching WebDAV off now really does switch it off, and back on restores everything. Repeat offenders are blocked, on by
default, taking care that the ordinary first request of every WebDAV program is never counted against anyone. Logins can be read
only, the size limit is a setting, and the administrator has a list of every login with when it was last used and a way to
remove one.
