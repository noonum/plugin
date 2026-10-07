# Analysis

Read this for outliers, breakdowns, comparisons, performance, and history. Every number comes from the Noonum connector's tools; scores are relative within one strategy and one read.

## Outliers

Work on the top 50: `get_strategy_companies` with `limit: 50`, plus `includeReasoning: true` with `limit: 20` for the top 20. Work out the median and quartiles of `linguisticBeta` across the 50. Flag a company when it is:

1. **Ranked high on buzz alone:** in the top 20 by conviction, with linguistic beta in the bottom quarter of the top 50.
2. **Included through an indirect role:** its `reasoning` describes a customer, buyer, user, or investor in the theme rather than a maker or supplier.
3. **In an isolated sector:** a sector only one or two companies share that does not fit the objective.
4. **Thinly backed:** only one or two `summaries`, or none from the past year.

Re-check each candidate with `get_strategy_company` before you report it. For each one give its rank, its linguistic beta against the top-50 median, and one line on why it looks off. If nothing stands out, say the top 50 looks coherent.

Only suggest a refinement (an exclusion phrase, an objective edit); never apply one from here.

## Sector, region, and size breakdown

- Base it on the top 50 and say so ("top 50 of 639"). The full list has a long tail scoring near zero.
- `sector` and `industry` labels come from the data provider and are not GICS.
- Region: by domicile (`domicile`, `domicileCode`) or by listing (`mic`, `exchange`); say which.
- Size buckets on `marketCap`: ≥$200B, $10–200B, $2–10B, <$2B, and unknown.

| Sector | Companies | Share | Top names |
|---|---|---|---|

Call out the largest buckets and anything the objective leads you to expect that is missing.

## Compare two strategies

- Read both with the same `limit` (50) and the same filter. When their saved filters differ, pass `exchangeRegion: []` to both, or the same explicit filter to both.
- Match companies by `id`, not ticker; the same company can show a different listing.
- Report:
  - overlap: the count, and its share of each strategy;
  - Jaccard similarity: overlap divided by the union;
  - for the overlapping companies, the rank on each side;
  - the top 10 unique to each, by rank;
  - the sector mix side by side, with the difference in percentage points;
  - performance side by side (below).
- Never compare raw scores across strategies.

## Performance

From `performance.metadata` on `get_strategy` or `list_strategies`:

| Field | Label |
|---|---|
| `returns` | 1-yr return |
| `annualizedReturn` | Annualized return |
| `variance` | Volatility (annualized) |
| `maxDrawdown` | Max drawdown |
| `sharpeRatio` | Sharpe ratio (no risk-free rate) |
| `stockCount`, `weekCount` | Companies in the basket, weeks covered |

- Say once what it is: a hypothetical conviction-weighted basket of the current top 50 over the past year. That is hindsight, not a track record.
- `performance` null means it has not been computed yet.
- There is no NAV series to chart or to compute other periods from; for the chart, give the dashboard link.

## History and versions

- `get_strategy_available_dates` lists the dates with a completed run for a version. Read a past run by passing that date as `asOfDate`.
- `list_strategy_versions` lists the versions, newest first, each with its objective, exclusions, waivers, and run state. The owner sees all of them; anyone else sees only the active one. Read an old version by passing `versionId`.
- To show what changed between two runs or versions, diff the top 50 by `id`: removed, added, and moved (rank before and after). Mention changes to the objective, exclusions, or waivers alongside.
