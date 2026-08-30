# Phase 161 - The Address Bar Problem

Visiting a customer domain's `/webmail` redirected the browser to the panel's own hostname. A white-label host should keep the customer's own domain in the address bar.

## From redirect to proxy

The original redirect was chosen for simplicity. We replaced it with a reverse proxy for `/webmail`, `/account` and `/controller`, taking care that sessions stay scoped to the customer's domain and that the real client address still reaches login protection and logging.

## Subdomain shortcuts

The `webmail.` and `account.` shortcuts now request certificate names for those hostnames, falling back gracefully if issuance for the full set fails. The `controller.` shortcut deliberately remains a plain redirect, keeping the admin entry point separate from customer certificates. Subdomain sites are excluded.

## Existing accounts

A scheduled sweep brings existing accounts up to date a few at a time.

## Verifying

On the development server, a full login flow on a test domain stayed on that domain, and a failed login recorded the real client address. The release was then deployed to all servers and verified. The Nginx Settings snippet needs to be re-applied once on servers already running; Phase 162 later automated that.

Tested (2543/2543).
