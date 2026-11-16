# Phase 285 - Emails That Look Like Ours

Everything the panel sends itself, the alerts, the sign-in notices, the login codes, the welcome and password emails, the
ticket updates and the reports, went out in the plain default layout the framework ships with: a bare product name, a
generic "Hello!", and "Regards". The subscriber welcome email next to them looked designed. These looked like an
afterthought, and they are some of the first things a customer actually sees.

## What changed

All of them now share one layout in the same style as the welcome email. A navy header carries the logo, a small
category (Security, Scheduled job, Support) and the headline, and the colour of the category shows how serious it is, so
a failed job reads differently from a clean scan before you open it. The facts that used to be loose lines of text now
sit in a card: a username and password, the time and address of a sign-in, the name and exit status of a job. A login
code is shown large enough to read at a glance, what a failed job printed sits in a dark box, and a support reply is
quoted. The footer says why you received the email and which server it came from.

Two smaller things matter more than they look. The order on the page now follows the order the email was written in, so
a note meant to come after a card comes after it. And every email has a proper plain-text version, for the people and
programs that read mail that way.

The messages an administrator writes and sends to a customer are wrapped in the same header and footer, so a hand-written
note arrives looking like it came from the same place as everything else.
