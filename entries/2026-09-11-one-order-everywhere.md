# Phase 226 - One Order, Everywhere

The fix from a few hours ago solved the ordering complaint exactly where it was raised — the
Domains page, and nowhere else. That was never really the scope of the problem. It was just the
first place someone happened to look.

## The same question, asked by a different page

Email Accounts' tabs are a different piece of UI entirely from the Domains page's table — different
controller, different template, built at a different point in the project. But underneath, they're
answering the exact same question: given this account's domains, in what order do I show them? Two
unrelated screens had converged on the same wrong answer to that question, independently, because
neither one had ever been told there was a right answer to converge on. Insertion order wasn't a
decision either page made — it was what was left once nobody decided anything at all.

Once that's visible, it stops looking like two bugs and starts looking like one fact that thirteen
different places in the codebase each happened to get wrong the same way, because none of them knew
about the other twelve.

## Naming the shared answer once

The fix that already existed inline on the Domains controller — sort by group, then by when it was
added within the group — wasn't wrong. It just lived in exactly one place, with no name, available
to exactly one caller. Thirteen places asking the same question needs one answer they can all reach,
not thirteen copies of the same three-line sort quietly drifting apart the next time someone tweaks
one of them without remembering the other twelve exist.

Pulling it onto the account itself — ask the account for its own domains "in order," rather than
asking for the raw list and sorting it yourself — turns "does this page also have the bug" from a
question you have to go looking for into one the account answers for you, every time, by
construction.

## What actually finished

Thirteen places that each independently arrived at the same wrong ordering now share one answer,
asked in one place. A page built next month asking the same question gets the right order without
anyone having to remember this conversation happened.
