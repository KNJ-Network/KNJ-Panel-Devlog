# Phase 143 - When the Fix Runs Clean and the Bug Doesn't Care

Phase 142's `des_key` fix was real — wrong math, confirmed empirically, corrected, released, and the
self-repair step confirmed to run successfully against the live Mail Only satellite. Then we reloaded
`/roundcube/` and got the exact same bare 500 back.

That's the uncomfortable case: a real bug, a real fix, a confirmed-clean run, and the symptom doesn't
move. Sent a second investigation pass in explicitly to rule the `des_key` theory back out rather than
re-confirm it, and it came back having exhausted static analysis — PHP version, FPM pool config,
MariaDB setup, package versions, all checked against both roles and all ruled out as the cause. What
was left pointed at Roundcube's own MySQL schema import, but confirming that meant reading actual log
output, and there is no log output reachable from source code alone.

No SSH on this stack either, on purpose — the whole point of this audit is treating it like a real
customer's server, and a real customer doesn't get to ask us to SSH in and look. So the honest move
wasn't a third guess, it was building the thing that lets us see the real error the way an actual
install would have to: a small, read-only diagnostic action (`roundcube-diagnostics`) that tails
PHP-FPM's global error log and Roundcube's own app log straight off whichever server is currently
running it, surfaced through a new "Roundcube Diagnostics" page under Mail. Existing precedent all the
way down — the read-only-action allowlist in the provisioning script, the mail-touching dispatch
pattern `Mail Queue` and `Mail Delivery Reports` already use so the read follows wherever mail
actually runs, the same role gate that keeps it off DNS Only.

Nothing about the underlying 500 is fixed yet. This is instrumentation, not a patch — the next step is
actually reading what it shows on the real Mail server and finding the real cause from there, not from
another round of static review.

Tested (2506/2506).
