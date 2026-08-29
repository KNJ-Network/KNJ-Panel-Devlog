# Phase 142 - A Webmail Key That Was the Wrong Length

Straight on from Phase 141: with the routing fixed, "Open Webmail" on a Mail Only satellite stopped
returning an access error and started returning a server error. The web server now reached Roundcube
correctly, and Roundcube itself was failing.

The obvious places showed nothing useful, because the failure was not being written down anywhere. Two
separate problems explained that. First, the key Roundcube uses to protect stored session data was being
generated one character short of the length it requires, so every request that needed it failed. The code
carried a comment claiming the generation always produced enough characters; it did not, and running the
generation repeatedly in isolation proved it. Second, the folder Roundcube writes its logs to had the wrong
permissions, so the real error had nowhere to go.

Both are fixed, the key check now verifies length rather than mere existence, and a server that already had
a short key repairs itself the next time its hostname or certificate is confirmed in Server Setup, so
nothing needs to be done by hand on existing installs.

The lesson: a comment that states a guarantee is not a test of it. The first thing to do with an
unexplained failure is make the real error visible.

Tested (2506/2506).
