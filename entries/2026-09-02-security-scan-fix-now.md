# Phase 173 - Not Every Warning Deserves the Same Button

Security Scan reports on its checks but never offered a way to fix them. We added "Fix Now", but different checks need different treatment, so there are three kinds of fix.

- **Auto** for checks that are safe to fix immediately.
- **Link** for settings owned by another page, so we do not create two writers for one setting.
- **Instructions** for changes where doing it automatically could cause harm, such as changes that could lock an admin out.

## Firewall

Enabling the default firewall policy is automatic only when the panel has confirmed that the ports an admin needs are already allowed. Otherwise the check falls back to instructions. The provisioning script verifies the same thing itself before making any change.

## Background work

One fix runs as a background job rather than inside the request, and the button says so.

## Verifying

2673/2673 passing. On a development server we broke each condition in turn and confirmed the right fix or refusal, and that administrative access stayed available throughout.

Tested (2673/2673).
