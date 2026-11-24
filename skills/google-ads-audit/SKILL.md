---
name: google-ads-audit
description: Audit a Google Ads account through Zenith and turn what it finds into a prioritized plan. Use when someone asks to audit an account, find wasted spend, or asks what is wrong with, or what to fix first in, a Google Ads account connected to Zenith.
---

# Audit a Google Ads account with Zenith

Zenith's tools read Google Ads and never change it. Every recommendation here is something the person does in Google Ads themselves, so never say a change was made, applied or scheduled.

## 1. Pick the account

Call `list_accounts`. Use the exact `customer_id` it returns, never one you guessed.

- One account: use it.
- Several, and the person named one: match the name. If the name fits more than one account, or none, ask which.
- None: say that no Google Ads account is connected yet and that one can be added in Zenith, under Access. Stop there.

## 2. Settle the goal and the period

The goal decides what counts as a problem:

| `goal` | Means | Signs in the request |
|---|---|---|
| `efficiency` | lower cost per conversion | "wasting money", "CPA is too high", "cut costs" |
| `scale` | more conversions | "grow", "more leads", "we have budget" |
| `revenue` | higher return on ad spend | "ROAS", "revenue", "sales value" |

If the request doesn't say, ask once. If the person would rather not say, use `efficiency` and tell them you did.

The period is `last_30` unless they say otherwise: `last_7`, `last_14`, `last_28`, `last_90`, `mom` (last month against the one before), `qoq` (this quarter against the last), `yoy` (the last 30 days against a year earlier), or exact dates in `date_start` and `date_end` (YYYY-MM-DD).

## 3. Run the audit

Call `run_account_audit` with `customer_id`, `date_preset` (or the dates) and `goal`. Add `campaign_type` (`SEARCH`, `PERFORMANCE_MAX`, `SHOPPING`, `DISPLAY`, `VIDEO`, …) only when they asked about one type.

The result draws its own card. Don't pass it to `render_data_table`, and don't copy every figure back into the answer.

## 4. Look closer where the audit points

Only where it helps answer the request:

- Keywords or search terms that spend without converting: `get_keyword_analysis` (add `campaign_id` to stay in one campaign).
- A campaign far behind the others: `get_campaign_performance`, with `sort_by` set to what the goal cares about (`cpa` for efficiency, `conversions` for scale, `roas` for revenue).
- What to do first for the goal: `get_optimization_suggestions` with the same `goal`.

## 5. Write the plan

1. **The account in two sentences:** the period, spend, conversions, and the change against the period before that matters most.
2. **What's wrong:** the critical findings first, then the warnings, each with its figure.
3. **What to do, in order:** for each action, what to change in Google Ads, the evidence from the results, and the estimated impact, labelled as an estimate.
4. **What to watch:** one or two figures to check after the changes, and when to check them.

## Reading the figures

- Money is in the account's currency, as the results give it.
- CTR, conversion rate and impression share are percentages: `ctr: 2.35` is 2.35%.
- Deltas are signed fractions: `0.12` is +12% against the period before. A result without `deltas` has no period before to compare with.
- A search term that spent without converting is a candidate negative keyword, not a certain one: it may be a brand term, or it may convert offline. Say so when you suggest it.
- An account with no data for the period is an answer, not a failure. Say it plainly.
