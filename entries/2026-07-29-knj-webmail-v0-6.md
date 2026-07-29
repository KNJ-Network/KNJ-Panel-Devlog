# Phase 28 - Rich Compose, and a Sanitizer That Had to Earn Its Keep

v0.6.0: HTML compose. The plain textarea is now a real editor — Bold, Italic, Underline, bulleted
and numbered lists, links — built without a framework behind a small toolbar.

The part that mattered wasn't the editor, it was what happens to what it produces before it goes
into an outbound message. A compose box is the one place in the panel where arbitrary typed HTML is
expected, so everything goes through a strict allowlist sanitizer rather than pattern matching.

Our own test suite caught the sanitizer's first real bug before it shipped: certain disallowed
elements were being stripped but their contents kept, when the contents should be removed too. Fixed
with a dedicated test.

Two smaller pieces came along with it. Sending now builds a proper multipart message — the
sanitized HTML plus a plain-text version — so older or stricter clients can render it. And Reply
now quotes the original message, using its plain-text body only, since the sender's HTML isn't
trusted enough to round-trip into our own editor.

Verified by hand on a test mailbox: hostile markup typed into the editor was removed from the
stored and delivered copies while formatting stayed intact, and the multipart structure was
confirmed on the delivered message.
