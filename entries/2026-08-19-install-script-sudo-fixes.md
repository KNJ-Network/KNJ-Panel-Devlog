# Phase 100 - Install Script Fixes, a Login Bug They Uncovered, and a Licence Gap

The install script was run on a genuinely fresh, disposable VM for the first time since the setup
wizard shipped, which is the real test that wizard was built for. It surfaced three separate bugs,
found live rather than guessed, each with its own fix.

## Three bugs behind one symptom

All three showed up as some variant of "asks for a password it can never satisfy", but they had
different causes:

1. **Switching to the service account from a root shell still demanded a password.** The
   privilege-switching tool consults its own policy even when the caller is already root. It was
   replaced with a tool that requires root but skips that authentication step.
2. **The login-mode variant doesn't work for an account with no interactive shell.** The first
   version of the fix was its own regression, since the service account deliberately has no login
   shell. Non-login mode with an explicitly set home directory fixed it.
3. **The trial-licence request ran before the permission it needed existed.** The install-time
   request ran earlier in the install than the step that grants the app its privileges. It now uses
   the OS machine identifier, which needs no privilege. The trade-off is that it changes on an OS
   reinstall, which is acceptable because the licence server continues an existing trial rather than
   issuing a second one.

The first verification run looked clean and the real one didn't because the earlier run was
backgrounded, so the failure was silent. Only the foreground, terminal-attached run surfaced it.

## A login bug this uncovered, and a fix that looked right but wasn't

With a working install, the next login landed on raw JSON instead of the dashboard. A background
status poll in the header, if it fired while logged out (for example from a stale browser tab), was
treated as an ordinary page visit and its URL was remembered as the post-login destination.

The first fix, declaring a JSON `Accept` header on the poll, read as correct and passed tests but
did nothing. Testing against the real deployed box showed it was still redirecting, because the
app's exception-handling rule replaced the framework's default JSON detection instead of adding to
it. The real fix restored the default alongside the existing rule, and was verified with a real
unauthenticated request returning a clean 401 with no redirect.

## Checking the rest of the install and update surface

The whole codebase was checked for the same pattern rather than just the one instance. The
self-update script had the same call shape in two places and was switched to the same fix, even
though the old pattern happened to work on the box tested, since it was not provably safe
everywhere. The DNS-only install path and FTP setup were clean, and a now-unused privilege rule the
install script used to write was removed.

## Also found: services reading as broken when they're just not installed yet

The install script doesn't install the mail, DNS, and FTP services without a domain, by design. But
the Service Status page and dashboard showed them as plain red "Inactive", indistinguishable from a
genuinely broken service. A third state, "Not set up yet", now points at Server Setup.

## DNS-only should never hold a licence

The install script already skipped the trial request for the DNS-only role, but the hourly licence
validation had no role check, and the licence page was reachable. Both now check the role: the
validation job returns immediately for DNS-only, and the licence page actions return 404 for that
role. Stale test records on the licence server were also reviewed and cleaned up.

## Verified

The full suite (1,489 tests) and code-style checks pass. Live, on three separate boxes: two clean
installs in a row on a snapshot-restored VM, a test server upgraded through its own upgrade script
with the login fix confirmed afterward, and a DNS-only test server upgraded and confirmed to make no
licence calls.
