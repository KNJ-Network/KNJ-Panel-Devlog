# Phase 253 - The Folder Inside the Customer's Reach

The second audit found the most significant issue of the day, and it came from a sensible design choice.

## Quarantine in the wrong place

When the malware scanner flags a file, it moves it into a quarantine area so it can be reviewed, restored
or deleted. That area lived somewhere convenient and easy to explain, but too close to the customer's own
files. It was reproduced in our own testing and fixed properly: quarantine now lives somewhere only the
system can reach.

## The same idea, everywhere

The wider lesson was that a check followed by a use is only as safe as the moment between the two. We
reviewed every place where privileged actions touch customer folders and made them robust against that
class of problem, then checked each change on the development server and cleaned up afterwards.
