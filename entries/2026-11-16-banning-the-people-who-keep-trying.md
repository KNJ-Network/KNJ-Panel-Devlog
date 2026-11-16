# Phase 278 - Banning the People Who Keep Trying

Reading the logs of our own server made one thing plain. Of the thousand or so attempts to log in over SSH in a day,
the protection that bans repeat offenders had caught almost none. It was working exactly as it had been set up. It
was set up for a kind of attack that hardly happens any more.

## What we found

The attempts are all automated, and most are brief: a bot tries a username that does not exist and hangs up, so it
never reaches the step the protection was watching. Nothing got in, because SSH on that server accepts keys only. But
FTP had no protection at all: we found a password guesser make well over a hundred attempts in a few minutes. And the
one protection we had for customer websites could, in theory, have banned a large website network's address and cut
off every visitor using it.

## What changed

**More ground covered.** Password guessing against FTP, mail sending and mailboxes; SSH attempts that never reach a
password; and scanners that probe websites for files that should never be public. Each protection blocks only the
ports it is there for, so someone guessing FTP passwords can still read a website, and a scanner loses the websites
and nothing else.

**Longer bans for repeat offenders.** Each time an address is banned again the ban doubles, up to a week.

**Safe for customers.** The whole risk of banning is banning the wrong people. A missing page, a broken link, a
mistyped address, mail refused as unwanted and a dropped connection never count, because large mail providers and
ordinary visitors do all of those every day. What does count is only what an attacker does. And an administrator can
list addresses that are never banned; the server itself, its sibling servers and the big website networks are always
on it.

**Tested before it was trusted.** We ran each rule against real attack logs from a live server first, and against
ordinary visitor traffic to be sure it never matched. Then we simulated attackers on a development server and checked
that every protection banned them, only on its own ports.

## What it will not do

It will not stop the attempts. A ban catches an address that keeps trying; a bot that tries twice and leaves is
never banned, and should not be. What keeps an attacker out is strong authentication. The bans keep the noise, and
the load, down.
