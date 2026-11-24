---
name: google-ads-report
description: Write a client-ready Google Ads performance report from Zenith's data, covering a period against the one before, what drove the change, campaigns to watch and next steps. Use for weekly or monthly reports, client updates, or "how did the account do" questions about Google Ads accounts connected to Zenith, for one account or several.
---

# Report on Google Ads performance with Zenith

Zenith reads Google Ads and never changes it. The report describes what happened and recommends; it never says a change was made.

## 1. Accounts and period

Call `list_accounts` and use the exact `customer_id` values it returns. For one account, ask which if more than one could fit. For a roll-up of several (an agency's clients, say), run step 2 for each.

| They say | `date_preset` |
|---|---|
| last week, the past 7 days | `last_7` |
| the past two weeks | `last_14` |
| the past 30 days (the default) | `last_30` |
| last month | `mom` |
| this quarter | `qoq` |
| against last year | `yoy` |

Exact dates go in `date_start` and `date_end` (YYYY-MM-DD).

## 2. Read the account

1. `get_account_dashboard`: spend, conversions, cost per conversion, ROAS, CTR, conversion rate and impression share, each against the period before, with daily trends. Add `campaign_type` when the report covers one type only.
2. `get_campaign_performance`: every campaign with its figures and a `performance_rating` (`top_performer`, `average` or `underperformer`). Sort by `spend` unless the report is about something else.
3. For Search accounts, when cost without conversions is part of the story: `get_keyword_analysis`.

Each of these draws its own card. Don't pass their results to `render_data_table`.

## 3. Write the report

- **Summary:** three bullets, with the account and the period in the first.
- **Key figures:** a table with each metric, this period, the period before, and the change.
- **What changed and why:** tie each large change to the campaigns behind it.
- **Campaigns to watch:** the top performers and the underperformers, each with one line on why.
- **Next steps:** two to four recommendations, each marked as a recommendation, with the evidence behind it.
- **About this report:** the period, the account, "Google Ads data read through Zenith", and that nothing in the account was changed.

For several accounts, open with one table of all of them (spend, conversions, cost per conversion and ROAS, each with its change), then a short section for each account.

## Writing for a client

- Use plain words, and say what CPA and ROAS mean the first time they appear.
- Ratios are percentages (`ctr: 2.35` is 2.35%), deltas are signed fractions (`0.12` is +12%), and money is in the account's currency.
- A change of a few percent on small numbers is noise. Say so rather than explaining it.
- A result without `deltas` has no period before to compare with (it had no spend or clicks). Say that instead of showing changes.
