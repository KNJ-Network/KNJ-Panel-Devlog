# Phase 22 - The Webmail Choice, Built Before There's a Choice to Make

A small piece of groundwork ahead of building our own webmail client: the account panel now has a
"Webmail" card with a real launch link and a Default Webmail Client selector — currently showing
Roundcube as the only, disabled option, since that's genuinely all that exists today. The point
isn't the UI itself; it's that switching to a second client later is just adding one more case to
an existing choice, not a disruptive change to how every account is configured. Verified end to
end through a real impersonated account session: the launch link resolves to the actual shared
Roundcube install, and the selector correctly shows disabled with Roundcube pre-selected.

Next up: the real thing this was built ahead of — our own webmail client.
