# Refinement

Read this when the user wants to change what a strategy includes. Only the owner can refine a strategy.

## Pick the lever

| Need | Lever | Cost | Confirm |
|---|---|---|---|
| region or market-cap view | saved filter (`filter_intent`) | free | no |
| remove a type of company | `exclusions` | 1 credit | yes |
| admit pre-revenue companies or distributors | `waivers` | 1 credit | yes |
| results off-target, or a wanted company is never retrieved | edit `objective` | 1 credit | yes |
| rank only US-listed companies | `mode: "us"` | 1 credit | yes |

The saved filter is covered in SKILL.md. Everything else changes what the engine retrieves or accepts, so it needs a new run.

## Writing exclusions

- 2–6 words describing what the company *is*, as a plain phrase: no parentheses, lists, or examples.
- No "exclude", no negation, no company names, nothing so broad it hits the core of the theme.
- If the objective explicitly asks for the companies to remove, change the objective as well; an exclusion alone would contradict it.
- Base them on the `reasoning` of the holdings to remove: name the pattern they share, not the companies.
- Start with 1–3 phrases; more can be added later.

Good: "memory chip manufacturers", "contract electronics assemblers", "semiconductor distributors".

Bad: "exclude memory makers" (prefix), "companies not making logic chips" (negation), "Micron and SK Hynix" (company names), "large technology companies" (too broad).

## Writing waivers

Short phrases naming a readiness stage or a role tier the verifier rejects by default: "pre-revenue companies", "distributors", "resellers". A waiver lifts a rejection; it never adds a role.

## Editing the objective

Use it when results are off-target, or when `get_strategy_company` returns 404 for a company the user expects (the search never retrieved it). Add the missing product, activity, or value-chain segment in plain words. Keep the "The strategy is to invest in companies that…" form and no company names.

## US-only mode

`mode: "us"` restricts the universe the engine ranks to US-listed companies. If the user only wants to see US listings, the saved filter `exchange_region: ["United States"]` does that for free.

## Steps for a paid change

1. `get_strategy`: read the current `objective`, `exclusions`, `waivers`, `mode`, and `active_version_id`, and check ownership (`is_owner` true). If the user is not the owner, explain and offer a copy (1 credit, confirmation).
2. Draft the change. **Exclusions and waivers must carry every existing phrase plus the new ones**, because `update_strategy` replaces the whole list. Show which phrases are kept and which are new. Show a new objective in full, word for word as you will send it, never a summary.
3. Preview the likely effect: name the current holdings the change should remove or admit, based on their reasoning, and say that ranks and buzz will be recomputed. It is an estimate until the run finishes.
4. Call `get_usage`, then confirm. State the cost ("1 credit, N left"), that it takes about 3 minutes, and that the strategy shows no results until the new run finishes, while the current results stay readable with `versionId` set to the current `active_version_id`. Options: apply and rerun, edit, cancel.
5. `update_strategy` with only the changed fields, then `submit_strategy` without `as_of_date`, then wait (see "Waiting for a run" in SKILL.md).
6. Diff the old and new top 50 by `id`: removed, added, and moved (rank before and after). Read the old one with the saved `versionId`.
