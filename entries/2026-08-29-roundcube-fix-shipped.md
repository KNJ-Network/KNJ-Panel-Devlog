# Phase 152 - Guarding What Should Never Have Needed Guarding

`array_first()` isn't roundcube-core's oldest code. It's a compatibility shim, written for a PHP that
didn't have the function natively, doing exactly what such a shim should: check if the real thing
exists, and only step in if it doesn't. Except this one doesn't check. It just declares itself,
unconditionally, at file scope, on line 308 of a bootstrap file that runs on every single request.
PHP 8.5 added the real `array_first()`. From that point on, this box's Roundcube install never had a
chance.

The fix is small on purpose. Wrap the declaration:

    if (!function_exists('array_first')) {
    function array_first($array)
    {
        ...
    }
    }

That's it. That's the whole bug, and that's the whole fix — one line added before, one line added
after, the body untouched. Getting to a fix this small took five releases of chasing something that
turned out not to be the bug at all.

Wrote it as an exact string match, not a line-number patch. The confirmed source — pulled straight
off this Roundcube build via the diagnostic from the last two releases — is the literal needle
`str_replace` looks for. If `roundcube-core` ever reformats this function in some future package
version, the match just won't fire, and the diagnostics page (which already dumps this exact context)
is still there to say so, instead of a blind sed silently mangling something it doesn't understand.
Verified locally against a byte-for-byte copy of the real source before it ever touched the live
server — the patch applies cleanly, produces valid PHP (`php -l` passes), and a second pass over an
already-patched file correctly does nothing.

One thing worth being honest about: `bootstrap.php` lives under `/usr/share`, not `/etc`. It's not a
conffile dpkg protects. The next `apt upgrade roundcube-core` on this box will silently overwrite this
edit with the stock, unguarded version — and the 500 would come back, silently, until someone hits it.
That's not a gap being papered over; it's why this patch is a self-repair inside `issue-panel-cert`
rather than a one-time manual fix. Cert renewal runs this action routinely. Any admin re-confirming
their hostname runs it too. The guard reapplies itself the next time either happens, no different from
how the FPM pool-config fix two releases ago handles the same kind of drift.

Four releases of the wrong bug, one release of the right one. The whole investigation, in order:
guessing at ownership from side effects, finding a real-but-separate `set -e` bug along the way,
finally reading an unsuppressed error instead of guessing, discovering the log itself was the thing
lying, and only then getting to read the actual fatal for the first time. None of the earlier steps
were wasted — the pool-config fix is real and would have mattered eventually regardless — but the
500 itself was never going to yield to anything except reading what PHP was actually saying.

Tested (2506/2506).
