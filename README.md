# Supplier Spend Analysis: Government of Canada Contracts (Power BI)

A procurement-style analysis of open government contract data: which suppliers hold the most contract value, how concentrated that value is, and which contracts are about to end.

**Data:** [Proactive Publication - Contracts over $10,000](https://open.canada.ca/data/en/dataset/d8f85d91-7dec-4fd1-8055-483b77225d8b), Government of Canada. Data downloaded: [DATE].
**Tools:** Power BI Desktop, Power Query (M), DAX. No external code or scripts.
**Scope:** contracts dated on or after 2022-04-01 with vendors whose postal code starts with "T" (Alberta). End dates measured as of 2026-10-06. 23,850 contracts, $6.66B total contract value, 4,099 suppliers.

## Questions answered
1. Which suppliers and commodity categories carry the most contract value?
2. How concentrated is that value (top-N share, Pareto, Herfindahl-Hirschman Index)?
3. Which contracts end within 90, 180, and 365 days, and how much value is up for renewal or re-sourcing?

## Dashboard
![Overview](images/01_overview.png)
![Concentration](images/02_concentration.png)
![Categories](images/03_categories.png)
![Expiring contracts](images/04_expiring.png)

## Key findings
- **Fragmented overall.** The top 10 suppliers hold 32.2% of contract value and the top 3 hold 13.8%. Overall vendor HHI is 146, far below the 1,500 threshold for an unconcentrated market. The largest supplier, Ledcor Highways, holds about $350M.
- **Concentrated inside specific categories.** Among the 20 largest commodity codes, HHI is above 2,500 in several. For example, code E199 has $462M across 42 vendors, an HHI of 4,445, and its top 3 vendors hold 94.5% of the value. Low overall concentration hides single-category dependency.
- **Contract value by type:** Services $3,557M, Construction $2,158M, Goods $693M, Uncoded $248M.
- **Data gap:** $320M (4.8%) of contract value has a blank, "0", or "#N/A" commodity code and is shown as "Uncoded".
- **Contract value by fiscal year (April to March):** 2022-23 $2.11B, 2023-24 $1.85B, 2024-25 $1.09B, 2025-26 $1.40B, 2026-27 $0.20B (year to date).
- **Renewal exposure.** $913M of contract value has an end date within 12 months of 2026-10-06: $92M in 0 to 90 days, $767M in 91 to 180 days, and $54M in 181 to 365 days. 1,267 contracts end within 180 days. $658M (72%) of the 12-month total has a March 2027 end date, and the largest watchlist contracts show March 31, 2027, the federal fiscal year-end. The data cannot say why end dates cluster there.
- **Largest expiring supplier:** KBL Projects holds $135.6M, 14.9% of the value ending within 12 months.

## Method
1. **Parameters** (`SinceDate`, `PostalPrefix`, `AsOfDate`) control scope and the as-of date without editing code.
2. **Cleaning in Power Query:** keep the needed columns, parse dates and currency tolerantly, filter to scope, drop rows with no vendor or a non-positive value, and normalize vendor names (case, punctuation, legal suffixes such as INC, LTD, LTEE) so spelling variants of one vendor combine. Blank, "0", and "#N/A" commodity codes become "Uncoded".
3. **Fiscal year** is derived on the April to March calendar. The current year is labeled "(YTD)".
4. **Model:** a `vendor_summary` table (value, share, rank, cumulative share) related to `fact_contracts` on the cleaned vendor name.
5. **Measures** in DAX: totals, top-N share, HHI, and value ending in 0 to 90, 91 to 180, and 181 to 365 days.
6. **Validation:** top 10 bars sum to 32.2% of total; fiscal-year bars sum to the total; expiry buckets sum to $913.06M; category bars sum to the total.

## Limitations
- `contract_value` is the total contract value across all years and amendments, including taxes, as defined in the dataset's schema. It is not annual cash spend. Standing offers and supply arrangements are reported as $0, so spend through them is understated.
- Only contracts over $10,000 are published.
- "Alberta" means the vendor's postal code starts with T. It does not mean the work was performed in Alberta.
- Vendor names are normalized by rules. Affiliated companies and joint ventures are not merged (for example ATCO Frontec and its joint venture are separate), so concentration can be understated.
- `delivery_date` is the contract period end date or delivery date. End dates may be extended later by amendment.
- Commodity codes use more than one scheme (goods codes, service codes, construction codes starting with 51). They are shown as coded, not relabeled.
- Individuals' names are excluded from the renewal watchlist using a keyword rule, which can miss or wrongly exclude a few records.

## Reproduce
1. Download the national contracts CSV from the dataset page above (it is large and is not stored in this repo).
2. Open `Supplier_Spend_Analysis.pbix`, then Home > Transform data. Point the `contracts` query at your downloaded file and adjust the parameters if you want a different scope.
3. Close & Apply. To rebuild from scratch, use the code in `powerbi/`.

## Repo layout
| Path | Contents |
| --- | --- |
| `Supplier_Spend_Analysis.pbix` | The report (contains the filtered, cleaned data) |
| `images/` | Dashboard screenshots |
| `powerbi/fact_contracts.m` | Power Query for the cleaned fact table |
| `powerbi/fnCleanVendor.m` | Vendor-name normalization function |
| `powerbi/parameters.md` | Parameters and their values |
| `powerbi/model.dax` | Calculated tables and columns |
| `powerbi/measures.dax` | All measures |

## Attribution
Contains information licensed under the Open Government Licence – Canada.
