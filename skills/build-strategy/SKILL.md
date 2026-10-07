---
name: build-strategy
description: Builds a new Noonum thematic investment strategy from a thesis, theme, article URL, or pasted text, runs it, and presents the ranked companies with the reasoning behind each. Use when the user wants to build, create, start, or run a new thematic strategy or theme basket ("build a strategy for...", "find companies benefiting from...", "turn this article into a strategy"). For existing strategies (holdings, analysis, refinement, factsheets, archive) use the strategy skill.
compatibility: Requires the Noonum connector (https://mcp.noonum.ai/mcp) signed in to a Noonum account. Each run spends one plan credit.
---

# Build a Noonum strategy

A strategy is a natural-language objective that Noonum turns into a ranked list of companies, each with the reasoning and evidence for its inclusion.

## Standing rules

- **Connector required.** All strategy data comes from the Noonum connector's tools. If they are missing or return an auth error, stop and tell the user to connect or sign in to the Noonum connector. Never produce strategies, companies, scores, or evidence from your own knowledge.
- **Grounding.** Every company, score, reason, and piece of evidence you show comes from a tool response in this conversation. Your own knowledge may explain what a company does, never why it is in a strategy.
- **Confirm before spending.** A run spends one credit. Get one confirmation before creating and running (step 4).
  - The user must see the full draft when they choose. If your choice tool can show a preview next to an option, use it and put the full draft in the preview of the option that runs it. Otherwise write the draft in your message and end with numbered options. A question that only mentions the draft does not show it.
  - Proceed only on a clear choice. If the reply is ambiguous, ask once more.
  - An explicit "go ahead" from the user counts as confirmation, including one given up front ("build it and run it"). Explicit user instructions take precedence over this skill.
- **Research, not advice.** Present results as research. No buy/sell/hold recommendations, price targets, or boilerplate disclaimers.
- **Out of credits.** On a 429 or `remaining` 0, give the refill date (`refills_at`). Never link to an upgrade or checkout page.
- **Dashboard link.** Every time you present a strategy, include `https://dashboard.noonum.com/strategy/{id}`.
- **Dates.** UTC, YYYY-MM-DD. Evidence `pubDate` is an ISO 8601 time in UTC; cite its date.
- **Wording.** Say "linguistic beta", never "beta" alone; to investors "beta" means market beta.

## 1. Get the thesis

If the user gave a thesis, a URL, or a name, use it (it may arrive after `ARGUMENTS:`). Otherwise ask for one. If the idea is too vague to scope ("something about energy"), ask one clarifying question, no more.

For a URL, read the page with whatever web access the environment has and distill its investment theme. Without web access, ask the user to paste the text or state the thesis; never guess a page's content from its URL. If the page has no investable theme, say so instead of forcing one.

## 2. Draft the name and objective

- **Name:** 2–5 words, descriptive, no "Strategy" suffix ("AI Infrastructure", "GLP-1 Supply Chain").
- **Objective:** one paragraph that starts with "The strategy is to invest in companies that…". Describe what the companies make, do, or sell and which parts of the value chain count. Keep the user's intent and scope. No company names (Noonum finds them), no negation, no bare sector labels.

Good: "The strategy is to invest in companies that manufacture solid-state batteries for electric vehicles, including suppliers of solid electrolytes, lithium-metal anodes, and cell production equipment."

Bad: "The strategy is to invest in energy companies like QuantumScape, but not oil producers." (a sector label, a company name, and a negation)

Put each constraint the user states in the right field, not in the objective:

| Constraint | Field |
|---|---|
| a type of company to remove | `exclusions`: 2–6 word phrases saying what the company *is* ("semiconductor distributors"); no "exclude", no negation, no company names |
| pre-revenue companies or distributors, which are rejected by default | `waivers`: "pre-revenue companies", "distributors" |
| US-listed only, a region, a market-cap range | `filter_intent`: a free display filter that never narrows what is scored |
| a universe of US-listed companies only | `mode: "us"` (rare; only when the user wants the ranking itself US-only) |

`filter_intent` is `{"exchange_region": [...], "domicile": [...], "market_cap": {"min": ..., "max": ...}}` in USD, with a null bound left open. Region labels: United States, Developed Markets, Emerging Markets, Frontier Markets, Europe, United Kingdom, EMEA, Asia (incl. AU/NZ), Asia (excl. AU/NZ), Japan, China, India, North America, Latin America, MENA, Africa. `exchange_region` also takes exchange MICs (XNAS); `domicile` also takes ISO country codes (US, JP). "US-listed" is `exchange_region: ["United States"]`.

## 3. Check for duplicates

Page through the Noonum connector's `list_strategies` tool with `scope: "all"` and `limit: 50`, raising `offset` by 50 until you have seen `total`. Compare names and objectives by intent, not wording.

- Classify each match by `public`: `true` is the public catalog, `false` is the user's own. Never call the user's strategy public.
- A close match becomes a free option at confirmation: open the existing strategy instead.
- The user's own match with `status` 0 is a draft that never ran: offer to run it instead of creating a duplicate.

## 4. Confirm once

Call `get_usage`, then show the draft as plain lines with a blank line between them, not a table or a code block: a `Name:` line, then an `Objective:` line with the full text you will send to `create_strategy`, word for word, never a summary. Add a line for exclusions, waivers, display filter, or US-only universe only when they are set. Then one line: "Running it costs 1 credit (N left this period) and takes about 3 minutes." On an unlimited plan (`remaining` null), say the plan is unlimited.

Options:

1. Create and run
2. Edit the draft
3. Open "{existing name}" instead (only when step 3 found a close match; free)
4. Cancel

With 0 credits left, replace option 1 with "Save as a draft" (`create_strategy` only, free) and give the refill date. On "Edit", ask what to change, redraft, and confirm again.

## 5. Create, submit, wait

1. `create_strategy` with `name`, `objective`, and only the fields that are set.
2. `submit_strategy` with the new strategy `id`. Leave out `as_of_date` so the run is live. Set `create_factsheet: true` only if the user asked for a factsheet.
3. Wait with `get_strategy_version`, using the `version_id` from the submit response:
   - If you can pause between calls, check about every 30 seconds for at most ~6 minutes.
   - Otherwise check once, tell the user it is running (about 3 minutes) with the dashboard link, and check again on their next message.
   - Stop on `completed: true` or `status` -2.

`status`: 0–95 running; -1 a temporary error, retried automatically; 100 done; -2 failed, and the credit is not charged. On -2, say so and offer a retry (which needs confirmation again).

| Error | Meaning | What to do |
|---|---|---|
| 429 | No credits left this period | Give `refills_at` from `get_usage`; the draft is saved |
| 423 | A run for this version and date is already in progress | Wait for it; do not resubmit |
| 409 | This login's email already belongs to another Noonum account | Ask the user to sign in to the connector with that account |
| 400 | Invalid input | Show the message, fix the field, retry |
| 404 | Strategy not found, or not owned by the user | Re-check the id |

## 6. Present the results

Call `get_strategy_companies` with `includeReasoning: true` and `limit: 15`.

Header: **{name}**, {total} companies, as of {`latest_run_date` from `get_strategy_version`}, followed by the dashboard link.

| # | Company | Ticker | Sector | Mkt cap | Conviction | Ling. beta | Mkt buzz |
|---|---|---|---|---|---|---|---|

- Ticker is `symbol`. Mkt cap is `marketCap` in USD, short form with a dollar sign (1.05T, 224B, 9.9B, 850M).
- Conviction is `convictionScore` and Ling. beta is `linguisticBeta`, both to two decimals. Mkt buzz is `marketBuzz`, a share of the strategy's buzz, as a percentage (0.1289 is 12.9%, 0.0007 is 0.07%).
- A missing value shows as —; never drop a row.

After the table:

- One line on why each of the top 5 is included, from its `reasoning`.
- 2–3 observations grounded in the table (the sector mix, a cluster of equipment makers, an inclusion that looks indirect).
- Once: scores are relative within this strategy. Conviction combines linguistic beta (how strongly the company's own text ties to the theme) with market buzz (its share of the theme's recent attention). Compare them only within this strategy.
- When you cite evidence, write "Provider, YYYY-MM-DD".

Close with what the user can do next: ask why a company is in, look for outliers, refine, export, or request a factsheet.
