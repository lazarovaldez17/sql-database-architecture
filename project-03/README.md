# Breaking Games: Holiday Game Plan

An analytics engineering project built on Breaking Games' real Shopify and Meta Ads exports — turning six scattered, disconnected data sources into a database, a reorder recommendation, and a Q4 ad spend plan a growing e-commerce business can actually act on.

Breaking Games is a tabletop game publisher selling direct-to-consumer through breakinggames.com. The data here covers that Shopify side of the business only — six months of sales, checkouts, site referral traffic, and Meta (Facebook/Instagram) ad performance from January–July 2025. The wholesale and Kickstarter sides of the business run through separate systems and aren't in scope.

## The Business Problem

Two people at Breaking Games are flying blind on decisions with real dollars behind them:

- **Mike, Head of E-Commerce**, needs to decide which of 117 products to reprint before the Nov 1 holiday cutoff — and his current process is highlighting a printed spreadsheet by hand, which works at 15 products and breaks at 117.
- Mike also has a Q4 budget review with Shari where he has to answer: **did the $37,200 spent on Meta ads work, and where should the next dollar go?** Right now nobody can connect ad spend to actual revenue, because the two live in completely disconnected systems.

This project builds the database and the analysis to answer both.

## The Data

All source files load into a single SQLite database, `breaking_games.db`:

| Table | What it holds |
|---|---|
| `shopify_sales` | Per-product sales summary — units, revenue, AOV (99 products) |
| `shopify_orders` | One row per line item per order (July 2025 only) |
| `shopify_checkouts` | Cart-level detail, anonymized customers (2016–2025) |
| `referrals` | Site traffic by referrer + landing page — pageviews, checkout sessions, conversion rate |
| `meta_campaigns` | 35 Meta ad campaigns — spend, impressions, clicks, results (Jan–Jul 2025) |
| `meta_ad_sets` | 8 ad sets — audience-targeting detail one level below campaigns |
| `dim_product` | Built in Project 2 — one clean row per product, with a slug for matching across tables |
| `fact_product_performance` | Built in Project 2 — the per-product summary mart (units, revenue, AOV, pageviews) |
| `dim_referrer` | Built in Project 3 — maps all 422 distinct referrer strings to a 6-channel taxonomy |

None of these tables arrive pre-joined. A meaningful chunk of this project is building the keys and lookup tables that connect them, and being explicit about where the connection is a clean join versus an estimate.

## Project Status

**Project 1 — Raw Data.** Loaded Breaking Games' six scattered exports into `breaking_games.db` and profiled the data: anonymized customers, mismatched date ranges across files, and a handful of bot-like checkout patterns flagged for later investigation.

**Project 2 — Database Design.** Built `dim_product`, the primary/foreign-key structure connecting product data across tables, and a top-25 product performance leaderboard. Delivered a reorder recommendation: ~10 products to print heavy (led by the Dwellings of Eldervale line), ~5 to print light, ~3 to stop printing — each with a defensible, checkable number behind it.

**Project 3 — Following the Money.** Built a 6-channel taxonomy for `dim_referrer` (Paid Social, Organic Search, Paid Search, Organic Social, Owned, Other) to sort 422 raw referrer strings, learned why Breaking Games' data can't support a formal attribution model (no UTM tags, no session-to-purchase link — correlation only, stated explicitly rather than overstated), and shipped a Q4 ad allocation plan: cut the campaign that burned $12,917 (a third of the Meta budget) with zero recorded results, and reallocate the freed $16,349 to the account's proven performers. Full write-up: [`PROJECT-3-FOLLOWING-THE-MONEY.md`](PROJECT-3-FOLLOWING-THE-MONEY.md) · plan detail: [`Q4-AD-ALLOCATION-PLAN.md`](Q4-AD-ALLOCATION-PLAN.md). See `Design-referral-table-process.pdf` for the full `dim_referrer` design narrative.

**Project 4 — Abandoned Carts.** *Upcoming.* Investigate 1,000+ abandoned carts, separate real shoppers from bot activity, and find the true abandonment rate.

**Project 5 — Holiday Dashboard.** *Upcoming.* Build the holiday dashboard and final presentation, with a bonus round of creative marketing ideas for Breaking Games to consider.

## Tools

SQLite via DB Browser for SQLite, and Claude as a query-drafting and planning partner throughout — including learning to separate "plan with Claude" from "implement with Claude" as two distinct conversations.

## Why This Repo Exists

Beyond the assignment, this is meant to demonstrate real analytics engineering judgment: designing schemas, being honest about what a dataset can and can't support (attribution vs. correlation, ~50–65% match rates being normal for messy real-world data), and turning a query into a recommendation someone can actually defend in a room.
