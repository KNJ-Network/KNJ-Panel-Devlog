# Phase 291 - One Place For Mail Settings

Setting up how the server handles mail meant visiting three pages: Mail Settings, Mail Relay and Mailserver Config. They are
all server-wide mail settings, they were all narrow columns of plain fields, and none of them said whether mail was
actually working. The page that did show whether Postfix and Dovecot were running only did so when mail ran on a second
server, never for the machine in front of you. Handing mail to another server, which rewrites every domain's mail address
and copies every mailbox's settings, was a single click with no explanation.

## What changed

The three pages are now four tabs of one: where mail runs, the limits and checks on incoming mail, the outgoing relay, and
how mail apps connect. A strip across the top shows the state of mail from any tab: whether the mail services are running,
how many messages are waiting and how long the oldest has been there, whether a relay is in use, and whether the reverse
DNS is right.

Every setting now has a sentence saying what it does and its default, numbers are shown in words next to the figure, and
the risky choices warn as they are made. Edits across the tabs are saved together from one bar, which says how many changes
are waiting and applies only the parts that changed. Handing mail to another server now lists what will happen before it
does. The relay gained a button that tries the connection and the login the way the mail server will, without sending
anything, and explains what a relay does to how other servers see your mail.
