# Phase 143 - When the Fix Runs Clean and the Bug Doesn't Care

Phase 142's key fix was real: the key length was confirmed, corrected and released, and the self-repair
step ran successfully against the Mail Only satellite. Reloading the webmail page still gave the same
bare 500.

That is the uncomfortable case: a real bug, a real fix, a clean run, and the symptom does not move.
Static analysis of the PHP version, pool configuration, database setup and package versions ruled out
the usual suspects. What remained pointed at Roundcube's own schema import, but confirming that meant
reading real log output, and none is reachable from source code alone.

These servers are deliberately managed the way a customer's would be, through the panel only. So the
answer was not another guess but a tool to see the real error the way an actual install would need to: a
small read-only diagnostic action that reads the tail of the PHP-FPM error log and Roundcube's own log
from whichever server is running it, shown on a new "Roundcube Diagnostics" page under Mail. It follows
existing precedent: the read-only action list in the provisioning script, the dispatch pattern Mail
Queue and Mail Delivery Reports use so the read follows wherever mail runs, and the same role gate that
keeps it off DNS Only.

This is instrumentation rather than a patch; the next step is reading what it shows.

Tested (2506/2506).
