# Phase 150 - The Log That Was Never Being Written

While tracking down a webmail failure, we found that fatal errors from the webmail worker pool were not being captured anywhere. Earlier changes in this investigation had been aimed at a logging problem that turned out to be a symptom rather than the cause.

The reason was simple: the pool configuration did not ask the process manager to capture worker error output, so a fatal error ended without leaving a trace. That meant ordinary diagnostics showed nothing even though the page was failing every time.

We fixed it in two places. The server setup now configures error capture for new installs, and the provisioning step repairs existing installs automatically the next time it runs. The diagnostics page also shows the relevant log so the next failure is visible.

The lesson: before debugging a failure, confirm the failure can actually be observed.

Tested (2506/2506).
