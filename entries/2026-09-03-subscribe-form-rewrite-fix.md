# Phase 192 - The Same Symptom, a Different Door

After the previous fix, the signup form still failed with the same browser error. It is tempting to assume the same bug, but several causes can look identical from the outside.

## A second cause

A direct request to the address returned "not found" rather than a permission error. A convenience shortcut that lets a domain's address act as an entrance to its panel was routing this new public page into the account area. The shortcut had a short list of exceptions for pages that must stay public, and this page type was not on it.

## The fix

We added the page to the exception list, and made already-configured domains pick it up automatically the next time their configuration is touched. We also made the header-attaching code explicit about error responses, instead of relying on framework behavior that could change.

## The lesson

"Still broken, same error" describes what the visitor sees, not the cause. Watching the failure again, down to a request that never reached the handler, was the only way to separate the two problems.

Tested (2780/2780).
