# Phase 268 - The Sign-Ups That Said Thanks

The newsletter box on our own website had gone quiet. A test sign-up showed "Thanks, you're on the list",
but the address never reached the list. The cause turned out to be two good decisions meeting each other.

## What happened

The panel refuses to let code on a hosted site open a connection to the panel's own ports. That rule exists
because hosted code could otherwise pretend to be anyone and dodge login limits, and it came out of an audit.
Our website is hosted the ordinary way, and its sign-up form passes each address to the panel from the site's
own server. From the day the rule arrived, every one of those requests was refused. The form, written to
never show a visitor an error, said "Thanks" anyway.

## What it cost

A handful of sign-ups over about five days. The form never recorded the addresses it failed to deliver, so
they cannot be recovered. That is the real fault, and it is ours: a form that always says thanks is a form
that can lose people silently.

## What changed

- The website now keeps any address it cannot deliver and sends it on as soon as the panel answers again. If
  it cannot even keep it, the visitor is told.
- The panel gives sites on the same server a narrow, safe way in: a route reachable only from the server
  itself that accepts a sign-up and nothing else. Whatever address the caller claims is thrown away, so the
  original protection is not weakened. It installs itself with every upgrade, so every customer server has it.
- The Subscribers guide now explains how to build your own form or post from your own code, what comes back,
  and how a site on the same server should do it.

The lesson is the one we keep relearning: a friendly message is not a delivery. Check the answer, and keep
what you could not deliver.
