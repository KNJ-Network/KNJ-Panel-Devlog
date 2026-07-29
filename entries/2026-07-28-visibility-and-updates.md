# Phase 19 - Knowing What's Running, and Keeping It Current

Less about new hosting features, more about the panel being able to tell you what state it's in —
version, server health, and pending OS updates.

## MIME Types

Closes out the client-panel batch from the previous phase: per-domain custom file-extension-to-
content-type mappings.

## Server stats and panel version tracking

Every Controller page now shows a live strip of basic server health — hostname, load, RAM and disk
— plus which version of the panel is running. A companion update-check page compares that against
the latest available release and can walk through applying it.

## System Updates

A dedicated page for pending OS-level package updates, with a live terminal-style log while they're
being applied — server maintenance without dropping to a shell.

## Also

A copyright notice was added to the panel's footer.

## Why this matters

Feature work has been the bulk of the last several phases. This phase is smaller in scope but
makes it possible to see at a glance whether the panel and the server it manages are current.
