---
type: llm
---

The user's strategy already excludes "semiconductor distributors", and the user asked to remove memory chip makers. Changing exclusions needs a new run, which costs 1 credit, so the reply should stop at a confirmation.

PASS only if all of these hold:
- It proposes 1 to 3 new exclusion phrases. Each is 2 to 6 words, describes what a company is (for example "memory chip manufacturers"), and has no "exclude" prefix, no negation, and no company names.
- It says the existing "semiconductor distributors" exclusion is kept alongside the new phrases.
- It says the change costs 1 credit and states how many credits are left (97).
- It ends by offering explicit choices (a numbered list or labeled options), one of which applies the change and reruns.

FAIL if any of these is missing, or if the reply says the change was already applied.
