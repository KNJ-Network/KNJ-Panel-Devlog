# Phase 307 - One Place For PHP

Everything about how the server runs PHP was spread over three sidebar entries, and two of them were the oldest-looking pages
left: a narrow form pinned to the left with the rest of the screen empty. Extensions, the thing people most often come to PHP
for, sat behind a link on a row of a different page. Worse, the largest upload was set twice, once for the web server and once
for PHP, and whichever was smaller quietly won. Raising one did nothing, and nothing said so.

## What changed

PHP is now a single page with tabs for versions, extensions, settings, workers and the accounts using each version, opening on
a summary and a short list of what needs attention. The first thing it flags is a customer still running a PHP version that
has stopped getting security fixes, with a one-click way to move them. Every version shows when its fixes end.

The upload limit is now one setting, on the web server page, and PHP follows it, so the two can no longer disagree. The web
server and Perl pages were restyled in the same way, with the settings in sections side by side and a plain explanation beside
each choice.
