# Phase 283 - One Page Fewer

The DNS menu had a Nameserver Report: a table that looked up every domain's public nameservers each time it was opened
and said whether they matched. It was useful once, and then the DNS Zones page learned to answer the same question
better, and the report became a slower copy of something already on the list.

## What changed

The report is gone. Each domain's status, and where its nameservers point if not here, is on the zone list, answered
instantly from a check that runs in the background rather than made to wait while every domain is looked up. A filter
shows the domains that are not pointing here, including those whose nameservers could not be found at all, and one button
re-checks them all. The nameservers a domain should point at are on the Nameservers page, where they are defined. The
old address redirects, so nobody with it bookmarked lands on an error.

Taking a page away is a good result for a control panel: there was nothing the report could do that the list could not,
so keeping both only meant two places to look and two that could disagree.
