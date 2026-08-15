# calaccess-open-data

Aggregated, analysis-ready extracts from California's CAL-ACCESS campaign-finance
export, produced by [calaccess-pipeline](https://github.com/anaajibicho-gif/calaccess-pipeline).

**Source:** California Secretary of State nightly export
(`campaignfinance.cdn.sos.ca.gov/dbwebexport.zip`). All of it is public record.
This repository is public so the data can be loaded over HTTP.

**Window:** filings dated 2025-11-01 onward (the gap left when the California
Civic Data Coalition's pipeline stopped updating).

## Files

JSON chunks under `json/` are the load format; Postgres pulls them over HTTP.


| File | Rows | What it is |
|---|---|---|
| `data/race.csv` | 269 | Races, with committees and money raised |
| `data/committee.csv` | 4,530 | Committees with money, their race, raised and spent |
| `data/committee_month.csv` | 26,519 | Monthly raised/spent per committee |
| `json/donor.*` | 315,076 | **Every** donor from outside the committee system, with the zip / employer / occupation from their most recent gift |
| `json/donor_recipient.*` | 475,737 | Every donor's complete giving record |
| `json/committee_donor.*` | 488,281 | Every donor to every committee |
| `data/vendor.csv` | 17,717 | Non-committee payees, with their undated share |
| `data/vendor_month.csv` | 39,305 | Monthly money received per vendor |
| `data/transfer.csv` | 40,368 | Money between committees, both directions |
| `data/ie_target.csv` | 5,975 | Independent spending by target and position |

## Reading these numbers

- Figures are **gross, counted at every hop**. A dollar moving donor → PAC →
  candidate is disclosed twice. `transfer.csv` is where that hop is visible.
- **Never add across categories.** Periodic and 24-hour reports re-disclose the
  same dollars.
- **23.9% of expenditure dollars carry no transaction date**, so monthly vendor
  and spending series cover the dated share only — `vendor.csv` carries an
  `undated` column stating how much is excluded.
- `committee.csv` mixes candidate-controlled and independent-expenditure
  committees; the source does not distinguish them.


## Two horizons

Donor tables span the **full export** — 13.7M contributions, 1980 to 2026 —
because the point is an individual's giving record across cycles. Everything
else (races, committees, vendors, transfers, independent expenditures) stays on
the **current publish window**, so every figure on a committee or vendor page
reconciles with the others on that page.

| Table | Horizon | Rows |
|---|---|---|
| `donor`, `donor_recipient` | all history, 1980→ | 1.73M / 3.48M |
| everything else | 2025-11-01 → | see above |

Mixing them would produce cards reading "raised since 1999 / spent since Nov
2025" with cash-on-hand matching neither. Read the horizon before quoting a
figure.

## Completeness

Donor tables are **uncapped**. An earlier version published only the top 10,000
donors — 91% of the money but 3% of the people — which is the wrong trade for
matching individuals against a voter file, where a $50 donor matters as much as
a $500,000 one.

"Every donor" means every donor in the publish window (filings from 2025-11-01).
Extending further back is a `PUBLISH_FROM` change and a pipeline rebuild.
