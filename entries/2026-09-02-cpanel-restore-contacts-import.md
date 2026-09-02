# Phase 179 - The Address Book Nobody Went Looking For

While testing a restore from a real-world backup, everything checked out except the saved webmail contacts. The restore feature had never looked for them, although a full-account backup does carry them.

## Scope

The panel supports two webmail clients. Contact import is built for the richer one, which is the default for restored accounts and matches the backup's data natively. It is skipped cleanly, not as an error, for the simpler client.

## Preserving identity

Source and destination do not share numeric ids, so each imported contact and group is looked up by something that identifies it, such as an email address or a name. That also makes a second run a no-op.

## Verifying

Unit tests use a real database with the same table shape as a real backup, including a name containing an apostrophe to prove the generated SQL is safe. On the development server, all contacts, groups and memberships from the test archive landed correctly, and a repeat run duplicated nothing.

Tested (2699/2699).
