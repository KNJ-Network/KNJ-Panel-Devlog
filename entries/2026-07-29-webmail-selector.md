# Phase 22 - The Webmail Choice, Built Before There's a Choice to Make

A small piece of groundwork ahead of building our own webmail client: the account panel now has a
"Webmail" card with a launch link and a Default Webmail Client selector — currently showing the
existing client as the only, disabled option, since that's all that exists today. The point isn't
the UI itself; it's that adding a second client later is one more case in an existing choice, not
a disruptive change to how every account is configured. Verified end to end through an
impersonated account view: the launch link resolves to the shared install and the selector shows
it pre-selected.
