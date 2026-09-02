# Phase 175 - The Password That Broke Its Own Config

A real customer restore surfaced a bug that only a real generated password could ever trigger: a
restored WordPress site came back with a raw PHP parse error on the live domain. The database itself
was fine. The password was fine. `wp-config.php` was not — and the thing that broke it was the very
code meant to fix it.

## A replacement string is not a plain string

`replaceDbDefine()` rewrites `wp-config.php`'s own `define('DB_PASSWORD', '...')` line to match a
newly generated password, using `preg_replace()`. That function takes three arguments — pattern,
replacement, subject — and it's easy to think of the replacement argument as inert, just text going
back in. It isn't. `preg_replace()`'s replacement string has its own tiny grammar: a `$` followed by
a digit means "insert capture group N," left over from its original job of doing find-and-replace
with backreferences. A generated password is sixteen-plus characters of genuinely mixed content, and
sooner or later one of them looks like `Ej$6EYx$b\uWQj<cJ>7_.aB` — perfectly fine as data, and
completely wrong the moment it's read as a replacement pattern. `$6` doesn't mean "the literal
characters dollar-six." It means "capture group 6," which this regex doesn't have, so it's silently
replaced with nothing. The password that landed in the file wasn't the one that was actually set.

## The fix isn't a workaround, it's a different function

`preg_replace_callback()` takes a closure instead of a string, and a closure's return value is used
exactly as written — no grammar, no backreferences, no interpretation. The fix was swapping one call
for the other, nothing more: the closure just returns the same `define(...)` line it always did, but
now that string reaches the file byte for byte.

## Testing something that only fails on the right input

A test that hands this function an ordinary alphanumeric password would pass before this fix and
after it — the bug is invisible unless the input actually contains a dollar sign followed by a digit.
The regression test reflects a real password from a real broken restore
(`Ej\$6EYx$b\uWQj<cJ>7_.aB`) via reflection against the private method directly, since the real flow
always generates one randomly and there'd be no reliable way to reproduce this from the outside. A
`php -l` syntax-lint of the resulting file is the final check — the actual failure mode in production
wasn't a wrong value, it was a broken file.

Tested, verified against the real archive that surfaced it in the first place.
