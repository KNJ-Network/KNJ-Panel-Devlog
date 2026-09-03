# Phase 189 - The Page That Grew Instead of Scrolling

Feedback raised two problems in webmail: a sidebar that drifted out of reach as messages piled up, and an address book that was empty in one place and full in another.

## Growing instead of scrolling

The page height was set as a minimum, so it grew past the screen instead of scrolling the message list on its own. We fixed the outer height to the screen and let nested sections shrink, so the sidebar stays in place.

## Folders on the Contacts page

The Contacts page handed the layout an empty folder list, so the folders disappeared along with the way back to the inbox. It now requests the list like every other page.

## The empty address book

On a setup where mail runs on a separate server, the contacts sync request was not recognized there and was redirected to a sign-in page. The sender counted that as a normal response, so no error appeared. We taught the receiving server to accept this request type, and the sender now asks for a structured response so a similar problem would fail clearly. Afterwards the address books matched on both sides.

Tested (2777/2777).
