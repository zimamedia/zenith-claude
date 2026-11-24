---
name: google-ads-query
description: Answer Google Ads questions that Zenith's ready-made analyses don't cover by writing a Google Ads Query Language (GAQL) query, running it with Zenith's execute_gaql tool and showing the rows as a table. Use for top search terms, ad group or ad performance, device, location or day-of-week breakdowns, conversion actions, or any other custom report on a Google Ads account connected to Zenith.
---

# Custom Google Ads reports with GAQL and Zenith

## First, check for a ready-made analysis

These answer the common questions in one call, with their own card. Use them when they fit:

| Question | Tool |
|---|---|
| How is the account doing? | `get_account_dashboard` |
| How are the campaigns doing? | `get_campaign_performance` |
| Which keywords or search terms waste money? | `get_keyword_analysis` |
| What's wrong with the account? | `run_account_audit` |
| What should change first? | `get_optimization_suggestions` |

Write a query for everything else.

## The steps

1. `list_accounts`: pick the account and use its exact `customer_id`.
2. Write the query (below). When unsure of a field, look it up before running the query.
3. `execute_gaql` with `query` and `customer_id`. It returns `{ data: [...] }`: one object per row, keyed by the fields selected (`metrics.cost_micros`).
4. `render_data_table` with the rows exactly as returned, a short `title`, and the same `customer_id`. Zenith gives the figures their units (money in the account's currency, rates as percentages) and names the account on the card. A table holds at most 500 rows.
5. Answer the question in a few sentences. Don't list every row: the table shows them.

## Writing GAQL

```sql
SELECT search_term_view.search_term, campaign.name,
       metrics.cost_micros, metrics.clicks, metrics.conversions
FROM search_term_view
WHERE segments.date DURING LAST_7_DAYS
ORDER BY metrics.cost_micros DESC
LIMIT 10
```

- One resource in `FROM`, no joins and no `SELECT *`. Fields of related resources, such as `campaign.name` in a query on `ad_group`, can be selected where the view allows it.
- Dates: `segments.date DURING LAST_30_DAYS` (also `LAST_7_DAYS`, `LAST_14_DAYS`, `LAST_MONTH`, `THIS_MONTH`, `YESTERDAY`), or `segments.date BETWEEN '2026-10-01' AND '2026-10-31'`. Always bound the dates of a query that selects metrics.
- Selecting a segment such as `segments.date`, `segments.device` or `segments.day_of_week` splits each row by it.
- `metrics.cost_micros`, `metrics.average_cpc` and the other `_micros` fields are millionths of the account's currency: divide by 1,000,000 when quoting one yourself.
- `metrics.ctr` and the other rates come back as fractions (0.0408 is 4.08%).
- Leave removed items out with `campaign.status != 'REMOVED'` (or `ad_group.status`, `ad_group_criterion.status`).
- Always `ORDER BY` what matters, and add a `LIMIT`.

Where to start:

| Question | `FROM` |
|---|---|
| Account totals | `customer` |
| Campaigns, and their devices or days | `campaign` |
| Ad groups | `ad_group` |
| Keywords and Quality Score | `keyword_view`, with `ad_group_criterion.keyword.text`, `ad_group_criterion.keyword.match_type` and `ad_group_criterion.quality_info.quality_score` |
| Search terms | `search_term_view` |
| Ads | `ad_group_ad` |
| Locations | `geographic_view` or `user_location_view` |
| Conversion actions | `conversion_action` |

## When a field is in doubt

- `get_reporting_view_doc` with a view (for example `search_term_view`) lists what it can select, with its segments and metrics. Without a view, it gives a guide to every view.
- `get_reporting_fields_doc` with field names says whether each can be selected, filtered or sorted, and lists enum values.
- `get_gaql_doc` gives the whole grammar.

If Google turns a query down, read what it said (an unrecognized field is named), fix that one thing and run it again. After two tries, tell the person what Google said.

GAQL only reads: no query changes an account.
