# Phase 275 - The Address That Showed Someone Else's Site

While testing the webmail address of one of our own domains, we found it did not work. Over plain HTTP it showed a
"Not Found" page. Over HTTPS, it opened a different customer's website, with a certificate warning. Both were ours to
fix, and the second one was the more serious.

## What went wrong

**The webmail and account addresses.** Every domain on a server gets a webmail address and an account address of its
own. Two things had gone wrong with them. The plain HTTP version asked the panel for the page using the wrong name,
so the panel did not recognise the request. And once a domain's certificate covered the new names, nothing switched
the HTTPS version on, so the address was left answered by whatever came first.

**The bigger problem behind it.** A server that hosts many sites needs to know what to do with a web address that none
of them claims: a mistyped name, an old DNS record, an address that has not finished being set up. Ours had no answer
for that, so such a request went to whichever site the server happened to list first, and that site's page and
certificate were shown to the visitor. As far as we can tell nobody's private data was exposed, because the page was that
site's own public one. But a customer's own visitors should never be shown a stranger's website because of a typo.

## What changed

**Addresses that nobody claims are now refused.** The connection is closed on plain HTTP and the secure handshake is
refused on the secure ports, so nothing is shown at all. A server that was set up by its IP address, which already
has a default of its own, is left as it was.

**The webmail and account addresses work.** Plain HTTP goes to the sign-in page or on to the secure address, and the
secure address opens with the domain's own certificate as soon as the certificate covers it. Domains that were
already set up are corrected automatically after an upgrade.

## A mistake of ours on the way

The first repair we wrote cut a domain's configuration off at the first of these addresses and so also removed whatever
came after it, including a redirect and the domain's own panel addresses. We caught it on our own development server
within minutes, restored it, and rewrote the repair so that it only ever touches the two blocks it is rebuilding.
There are tests for exactly that layout now, because on a customer's server it would have been much worse.
