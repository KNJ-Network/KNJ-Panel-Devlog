# Phase 256 - The Page That Showed Zeros Politely

The Traffic & Visitors page had been empty for weeks, while the same site was gaining subscribers every day.
Nothing was reporting a problem, which was the problem.

## Allowed to look, not allowed to read

The page works out visits by reading each domain's web server log. After the logs were rotated, the panel
no longer had permission to read some of them, and the code treated a refused read the same as an empty file,
so the page cheerfully showed zero requests. Security hardening that arrived in the same stretch made it a
little worse.

## Three small changes

The permissions are now set up so the panel can always read the logs, existing logs were brought into line,
and the install, upgrade and automatic-deploy paths all apply the same steps, pinned by tests. If a log
exists but cannot be read, the page now says that plainly instead of showing a quiet month.
