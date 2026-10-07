---
name: strategy
description: Works with existing Noonum thematic investment strategies. Lists the user's strategies and the public catalog, shows holdings, explains why a company is or is not included with its evidence, finds outliers, breaks down sectors, regions and sizes, compares strategies, reports performance, refines exclusions, waivers, objectives and filters, gets factsheets, exports CSV, and archives or revives strategies. Use whenever the user names a Noonum strategy or asks about "my strategies", public strategies, holdings, or "why is X in it". Not for building a new strategy (use build-strategy).
compatibility: Requires the Noonum connector (https://mcp.noonum.ai/mcp) signed in to a Noonum account. Reruns and refinements spend one plan credit each.
---

# Work with Noonum strategies

## Standing rules

- **Connector required.** All strategy data comes from the Noonum connector's tools. If they are missing or return an auth error, stop and tell the user to connect or sign in to the Noonum connector. Never produce strategies, companies, scores, or evidence from your own knowledge.
- **Grounding.** Every company, score, reason, and piece of evidence you show comes from a tool response in this conversation. Your own knowledge may explain what a company does, never why it is in a strategy.
- **Confirm only what spends a credit, changes existing results, or starts a factsheet:**
  - changing the objective, exclusions, waivers, or mode and rerunning;
  - any run that is not free (see Rerun);
  - archiving a strategy;
  - generating a factsheet, or force-regenerating one.

  No confirmation for reads, filter changes (read parameters or the saved filter), rename, tags, or reviving.
- **How to confirm.** The user must see what they are approving (the change, the cost, what it affects) when they choose. If your choice tool can show a preview next to an option, use it and put that in the preview of the option that applies it, as plain lines, not a table. Otherwise write it in your message and end with numbered options. A question that only mentions the change does not show it. Proceed only on a clear choice; if the reply is ambiguous, ask once more. An explicit "go ahead" from the user counts as confirmation. Explicit user instructions take precedence over this skill.
- **Ownership.** `get_strategy` returns `is_owner` false when the user is not the owner. Only the owner can update, submit, archive, revive, or force a factsheet; for anyone else those calls return 404. For someone else's public strategy, offer to make the user's own copy (same name, objective, exclusions, and waivers). The copy has to run, so it costs a credit and needs confirmation.
- **Research, not advice.** Present results as research. No buy/sell/hold recommendations, price targets, or boilerplate disclaimers.
- **Out of credits.** On a 429 or `remaining` 0, give the refill date (`refills_at`). Never link to an upgrade or checkout page.
- **Dashboard link.** Every time you present a strategy, include `https://dashboard.noonum.com/strategy/{id}`.
- **Dates.** UTC, YYYY-MM-DD. Evidence `pubDate` is an ISO 8601 time in UTC; cite its date. Cite evidence as "Provider, YYYY-MM-DD".
- **Wording.** Say "linguistic beta", never "beta" alone; to investors "beta" means market beta.

## 1. Find the strategy

- A UUID, or a dashboard URL ending in one, goes straight to the Noonum connector's `get_strategy` tool.
- A name: page through `list_strategies` with `scope: "user"` (`limit: 50`, raise `offset` until you have seen `total`), then `scope: "public"`. Match by name and objective. If several match, ask which one.
- "My strategies" is `scope: "user"`; "public strategies" or "the catalog" is `scope: "public"`. The catalog is long: show the first page, say how many there are, and offer to narrow by theme.

| Name | Status | 1-yr return | Max drawdown | Link |
|---|---|---|---|---|

- Status from `status`: 100 Ready, 0 Draft (never run), 1–95 Running, -1 Retrying, -2 Failed.
- 1-yr return is `performance.metadata.returns` and max drawdown is `performance.metadata.maxDrawdown`, both as percentages; — when `performance` is null.

## 2. Holdings

Call `get_strategy_companies`. It ranks by `convictionScore`, highest first.

- Show 15–20 rows; fetch `limit: 50` for analysis. Use `includeReasoning: true` only with `limit` 20 or less.
- Header: name, `total` companies, as-of date, link. The as-of date is `latest_run_date` from `get_strategy_version` (with the strategy's `active_version_id`), or the `asOfDate` you passed.
- Table: `# | Company | Ticker | Sector | Mkt cap | Conviction | Ling. beta | Mkt buzz`. Ticker is `symbol`; Mkt cap is `marketCap` in USD, short form with a dollar sign (1.05T, 224B, 9.9B); Conviction and Ling. beta to two decimals; Mkt buzz is `marketBuzz`, a share of the strategy's buzz, as a percentage (0.1289 is 12.9%, 0.0007 is 0.07%).
- A missing value shows as —; never drop a row.
- If the strategy has a saved `filter_intent` with values, the list is already filtered: say so. Pass `exchangeRegion: []` to see everything.

## 3. Scores are relative

- `convictionScore` (0–1) combines `linguisticBeta` (how strongly the company's own text ties to the theme) with `marketBuzz` (its share of the theme's attention among the companies returned).
- Compare scores within one strategy only. A filter recomputes buzz and conviction over the companies it keeps.
- Place a score in the strategy's own distribution: "linguistic beta 0.03 vs a top-50 median of 0.18". Never use fixed cut-offs such as "below 0.2 is weak".
- Across strategies, compare overlap and ranks, never raw scores.

## 4. Tasks

### Why is it in, or not in

Get the company id from the holdings (`id`) or from `search_companies` (name or ticker; ask if ambiguous). Then call `get_strategy_company`.

- `isIncluded: true`: give its rank and scores if you have them, the `reasoning`, and the 3–5 newest `summaries` as "Provider, YYYY-MM-DD: text". Sort by `pubDate` descending before selecting them; the response is not necessarily newest first.
- `isIncluded: false`: the search found it but the verifier rejected it; give the `reasoning`. If the ground is readiness (pre-revenue) or role (distributor), a waiver may admit it.
- 404: the strategy's search never retrieved it. Only an objective change can bring it in.
- Included but missing from the list you showed: the saved filter may be hiding it.
- To keep companies like it out, the lever is an exclusion that names the type of company, never the company itself (see `references/refinement.md`). It needs a rerun, and only the owner can change the strategy.

### Filtering (free, no confirmation)

- **One-off view:** pass read parameters to `get_strategy_companies`: `exchangeRegion` (region labels or MICs), `domicile` (region labels or ISO country codes), `marketCapMin` / `marketCapMax` in USD. Any one of them replaces the whole saved filter for that read, so pass every part you want kept.
- A run dated before 2025-01-01 has no market caps (`marketCap` is null on every row), so a market-cap range is not applied to it: say so.
- **Sector or industry:** no parameter filters these, and they cannot be saved as a default. Page through with `limit: 100` and keep the rows whose `sector` or `industry` matches, until you have the rows you need or reach `total`. The labels come from the data provider, not GICS: say which labels you matched. Show each company's rank in the full list, and say the scores are those of the full list, not recomputed for the subset.
- **Lasting default:** `update_strategy` with only `filter_intent`, sending the whole object: `{"exchange_region": [...], "domicile": [...], "market_cap": {"min": ..., "max": ...}}`; `null` clears it. No rerun. Owner only; on someone else's strategy, use a one-off view instead. Then show the filtered holdings.
- Region labels: United States, Developed Markets, Emerging Markets, Frontier Markets, Europe, United Kingdom, EMEA, Asia (incl. AU/NZ), Asia (excl. AU/NZ), Japan, China, India, North America, Latin America, MENA, Africa. "US-listed" is `exchangeRegion: ["United States"]`; "US companies" by incorporation is `domicile: ["US"]`.

### Factsheet

Call `get_strategy_factsheet`.

- `ready`: give the `url`; it expires after 5 minutes (call again for a fresh one). If the environment can download files, save the PDF and give its path.
- `generating`: wait, following the waiting rule below.
- `not_requested`: say there is none yet and offer to generate it. Call `request_strategy_factsheet` only once the user chooses to, then wait.
- `unavailable`, or a 400 on request: the run is dated before 2025-01-01 and has no factsheet.
- `force: true` regenerates an existing factsheet: owner only, needs confirmation.
- Never build a factsheet yourself from holdings or metrics.

### Export

CSV columns: `rank,name,symbol,mic,isin,sector,industry,domicileCode,marketCap,convictionScore,linguisticBeta,marketBuzz`. Quote fields that contain commas.

- If you can write files, page through every company with `limit: 100`, write `{strategy-name}-{as-of-date}.csv`, and give its path.
- Otherwise put the top 100 in a CSV code block and say the full list is on the dashboard.
- Say whether the saved filter was applied.

### Archive and restore

- `archive_strategy` needs confirmation. It can be undone with `revive_strategy` for 30 days (`permanent_delete_at`).
- `list_archived_strategies` finds archived strategies. `revive_strategy` needs no confirmation; 410 means it is permanently deleted.
- Clean-up: list the candidates (never-run drafts, failed runs, obvious tests or duplicates) in one table and ask once: archive all of them, pick which, or cancel.

### Rerun and point in time

- `submit_strategy` without `as_of_date` reruns the current draft live. If a completed run already exists for that version and date, it is returned and no credit is spent (do not pass `force`). Otherwise it costs 1 credit and needs confirmation.
- A past `as_of_date` makes a historical run: 1 credit, confirmation, and no factsheet if it is dated before 2025-01-01. Keep the default `priority` 5; below 5 is batched and can take hours.
- Read a past run by passing `asOfDate` to the read tools.

### Waiting for a run

Use `get_strategy_version` with the `version_id` from the submit response. If you can pause between calls, check about every 30 seconds for at most ~6 minutes. Otherwise check once, tell the user it is running (about 3 minutes for a live run) with the dashboard link, and check again on their next message. Stop on `completed: true` or `status` -2.

### More

- Outliers, sector/region/size breakdowns, comparing strategies, performance, history and versions: read `references/analysis.md`.
- Exclusions, waivers, objective edits, US-only mode: read `references/refinement.md`.

## 5. Errors

| Error | Meaning | What to do |
|---|---|---|
| 400 | Invalid input: a malformed id, a future date, a factsheet for a run dated before 2025-01-01 | Show the message and fix the input |
| 404 | Not found, not visible to this user, a write by a non-owner, or no completed run yet | Check the id and owner; check the run state |
| 409 | This login's email already belongs to another Noonum account | Ask the user to sign in to the connector with that account |
| 410 | Archived more than 30 days ago and permanently deleted | It cannot be revived |
| 423 | A run for this version and date is already in progress | Wait for it; do not resubmit |
| 429 | No credits left this period | Give `refills_at` from `get_usage` |
| `status` -2 | The run failed; the credit is not charged | Say so and offer a retry, which needs confirmation |
