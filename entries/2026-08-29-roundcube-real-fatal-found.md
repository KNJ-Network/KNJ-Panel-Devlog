# Phase 151 - Array First, Finally Last

With error capture enabled, the real error finally appeared: the bundled webmail component declares a helper function that newer PHP versions now provide natively, which stops the page from loading. It was unrelated to the file permission theories we had been testing.

Before changing anything we added a read-only diagnostic to confirm the exact situation, then moved on to a fix that survives future package updates.

The lesson: read what the runtime is actually reporting before theorising about causes.

Tested (2506/2506).
