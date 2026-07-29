# Phase 28 - Rich Compose, and a Sanitizer That Had to Earn Its Keep

v0.6.0: HTML compose. The plain textarea is now a real editor — Bold, Italic, Underline,
bulleted and numbered lists, links — built the same way as everything else in this client:
no framework, just a `contenteditable` div and `document.execCommand` behind a small
toolbar.

The part that actually mattered here wasn't the editor, it was what happens to what it
produces before it goes anywhere near an outbound message. A browser's `contenteditable`
will happily hand back whatever markup it feels like, and a compose box is the one place
in this whole panel where arbitrary user-typed HTML is expected — which makes it the one
place a naive implementation would ship an XSS hole straight to whoever receives the mail.
So everything typed goes through a DOM-walking allowlist (`HtmlEmailSanitizer`) rather than
a regex: strip anything not on the list, strip every attribute except a checked `href`.

Own test suite caught the sanitizer's first real bug before it shipped: the initial version
handled every disallowed tag the same way — unwrap it, keep the text inside. Right instinct
for a stray formatting tag a user might paste in from somewhere; wrong for `<script>`,
where the whole point of removing the tag is removing what's inside it too. A test built
specifically to check `<script>alert(1)</script>` was actually gone — not just the tag,
the text — caught it failing with the *text* `alert(1)` sitting there inert but present.
Fixed by giving genuinely dangerous tags (`script`, `style`, `iframe`, `object`, `embed`,
`noscript`) their own path: removed whole, not unwrapped.

Two smaller pieces came along with it. Sending now builds a real `multipart/alternative`
message — the sanitized HTML plus a plain-text version derived from it, not an HTML-only
part that older or stricter clients can't render. And Reply now actually quotes the
original message for the first time — deliberately from its plain-text body only, never
its raw HTML, since that HTML came from whoever sent it and was never meant to be trusted
enough to round-trip back into our own editor.

Verified by hand on the real mailbox: typed bold text plus a `<script>alert(1)</script>`
and an `<img onerror>` straight into the live editor, saved it as a draft, and read the
stored message back over IMAP — script and `onerror` both gone, bold intact. Sent the same
message for real and read the delivered copy's `body_html` and `body_text` back over
IMAP too, confirming the multipart structure actually went out, not just rendered
correctly in this client's own inbox view.
