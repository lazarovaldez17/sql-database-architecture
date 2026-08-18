# Breaking Games — SQL & Database Architecture Externship

Turning six scattered exports from a tabletop games publisher into a queryable analytics database — and using that database to answer the question stalling the team: **what's actually working?**

This repo is a portfolio project built during a SQL / database-architecture externship. All data is real Breaking Games behavior with customer identity anonymized (see [Data Caveats](#data-caveats)).

---

## Mission

Connect Breaking Games' six raw exports into a single analytics database, use it to identify what's actually driving revenue, and turn that into a Q4 Holiday Game Plan the team can act on.

Breaking Games sells through several channels (hobby stores, online retailers, conventions, Kickstarter), but this project scopes to the **direct-to-consumer Shopify data** — the path from ad click to purchase on breakinggames.com. Wholesale and Kickstarter run through different systems and are out of scope.

## The Business Context

Breaking Games has 117 products across three broad buckets:

| Category | Examples | Buyer | Price Range |
|---|---|---|---|
| Strategy | Dwellings of Eldervale, Coaster Park | Dedicated tabletop hobbyists | $40–$200+ |
| Party / Light | POOP, Sparkle\*Kitty, We're Doomed! | Casual / gift buyers | $10–$20 |
| Family | The Game of 49 | Multi-generational play | $20–$40 |

Most revenue rides on a small subset of titles — roughly 20% of products drive about 80% of the money (the long tail). Part of this project's job is figuring out *which* 20%.

## Repository Contents

Six raw exports, currently ungoverned by any shared key:

| File | Rows | Cols | What it holds |
|---|---|---|---|
| `meta_campaigns.csv` | 35 | 19 | Meta (Facebook) ad performance, rolled up per campaign |
| `meta_ad_sets.csv` | 8 | 19 | Meta ad performance, rolled up per ad set |
| `referrals.csv` | 8,712 | 14 | Site traffic by referrer + landing page, with conversion/bounce/cart rates |
| `shopify_checkouts.csv` | 1,367 | 32 | Checkout-level detail: customer, line items, discounts, shipping, risk level |
| `shopify_orders.csv` | 254 | 12 | Completed orders, one row per line item |
| `shopify_sales.csv` | 99 | 10 | Product-level sales summary (net items sold, gross/net sales, returns) |

<details>
<summary>Column reference (click to expand)</summary>

**meta_campaigns.csv / meta_ad_sets.csv**
`reporting_starts, reporting_ends, campaign_name / ad_set_name, delivery_status, results, result_indicator, reach, frequency, cost_per_result, spend_usd, impressions, cpm_usd, link_clicks, shop_clicks, cpc_usd, ctr_link, clicks_all, ctr_all, cpc_all_usd`

**referrals.csv**
`referrer, landing_page, pageviews, added_to_cart_rate, pageviews_per_session, conversion_rate, bounce_rate, checkout_sessions, pageviews_previous_year, added_to_cart_rate_previous_year, pageviews_per_session_previous_year, conversion_rate_previous_year, bounce_rate_previous_year, sessions_that_completed_checkout_previous_year`

**shopify_checkouts.csv**
`name, email, customer_id, customer_name, financial_status, fulfillment_status, accepts_marketing, currency, subtotal, shipping, taxes, total, discount_code, discount_amount, shipping_method, created_at, lineitem_quantity, lineitem_name, lineitem_price, lineitem_sku, lineitem_discount, billing_city, billing_province, billing_country, shipping_city, shipping_province, shipping_country, payment_method, refunded_amount, vendor, risk_level, source`

**shopify_orders.csv**
`day, sale_id, order_name, product_title, gross_sales, discounts, returns, net_sales, shipping_charges, return_fees, taxes, total_sales`

**shopify_sales.csv**
`product_title, product_vendor, product_type, net_items_sold, gross_sales, discounts, returns, net_sales, taxes, total_sales`

</details>

## Data Caveats

- **Customer data is anonymized on purpose.** `shopify_checkouts.csv` replaces real names, emails, and addresses with placeholders like "Customer 001." Products, geography, and behavior are real; the people are not.
- **The files don't share a time window.** `shopify_orders.csv` covers July 2025 only. The Meta ad files run Jan–Jul 2025. `shopify_checkouts.csv` reaches back to 2016. Don't assume dates line up across files.
- **A few "customers" behave like bots** — adding the same item repeatedly in odd quantities. Flagged for investigation, not yet resolved in this repo.

## Externship Roadmap

- [x] **Project 1 — Raw Data Intake.** Get all six sources into a real, queryable database.
- [x] **Project 2 — Database Design.** Design a schema from scratch, connect the tables, and trace one customer's journey from ad click to purchase.
- [ ] **Project 3 — Ad Attribution.** Connect ~$37K of Facebook ad spend to actual revenue; recommend where Q4 dollars should go.
- [ ] **Project 4 — Abandoned Cart Investigation.** Separate real shoppers from bots across 1,000+ abandoned carts to find the true abandonment rate.
- [ ] **Project 5 — Dashboard & Holiday Game Plan.** Build a holiday dashboard and present a data-backed Q4 plan. ([Sample deliverable](https://smitabudhiaextern.github.io/extern-html-pages/dashboard-sample.html))

*Checkboxes reflect what's actually implemented in this repo — update as later projects land.*

## Database Design (Project 2)

### Why the raw files don't connect

A customer clicks a Facebook ad → lands on a product page → adds to cart → buys. That journey is scattered across four files, and none of them share an ID:

| Stage | Table | What we get | Where the trail breaks |
|---|---|---|---|
| 1. The ad | `meta_campaigns` | Spend + clicks, rolled up per campaign | No shared code linking a click to a site visit |
| 2. The visit | `referrals` | Referrer + landing page + conversion rate | No customer identifier captured |
| 3. The cart | `shopify_checkouts` | First customer identifier (email) | Nothing ties the cart back to the visit |
| 4. The order | `shopify_orders` | Completed sale detail | No email or cart reference back to checkout |

Product names compound the problem — the same game shows up as a URL slug (`dwellings-of-eldervale`) in referrals, a title with a variant suffix (`Dwellings of Eldervale 2nd Edition: Standard\| Standard`) in checkouts, and a clean title in orders.

### The fix: `dim_product`

A dimension table — one row per product, with a stable synthetic key every other table can reference:

| Column | Type | Why |
|---|---|---|
| `product_id` | INTEGER (PK, autoincrement) | Stable, database-generated identifier |
| `product_title` | TEXT | Full product name as it appears in Shopify |
| `product_slug` | TEXT | Lowercase, hyphenated version of the title, used to match against referral/landing-page data |
| `category` | TEXT | Strategy / Party / Family / Other |

**Rule of thumb:** anything intrinsic to the product goes in `dim_product`. Anything that varies by sale, customer, or date does not — that belongs in a fact table (e.g. `shopify_sales`, which keeps growing one row per event).

### JOIN cheat sheet

- **INNER JOIN** — only rows that match on both sides. Use when you only want data that exists in both tables.
- **LEFT JOIN** — every row from the left table, `NULL` on the right where no match exists. Use to find what's missing (products without sales, orders without a checkout match, etc.) via `LEFT JOIN ... WHERE right.col IS NULL`.
- Past 3 tables, chain JOINs one at a time with clear aliases (`dp`, `ss`, `r`), and consider whether a summary table would be cleaner.

### Realistic match rates

Slug- and title-based joins on real-world data don't hit 100% — and shouldn't be expected to:

| Join | Match rate | Why it's not higher |
|---|---|---|
| `dim_product` → `shopify_sales` (FK) | 100% | Built directly from these rows |
| `dim_product` → `referrals` (slug match) | ~52–68% | Auto-generated URL slugs add edition words / drop punctuation unpredictably |
| `dim_product` → `shopify_orders` (exact title) | ~64% | 92 of 254 order rows have a blank `product_title` (shipping/summary rows) |
| `dim_product` → `shopify_checkouts` (line-item name) | ~52% | Checkouts store `lineitem_name`, not a clean title — many carry price/edition tags |

A 50–65% match is normal for messy real-world tables. When non-matches share a clear, nameable cause (blank titles, edition suffixes), that's the accepted edge of real data — not a broken query. Only dig deeper if the misses look random.

## Getting Started

1. Open the six CSVs in [DB Browser for SQLite](https://sqlitebrowser.org/) (or your SQL client of choice) and import each as a table.
2. Build `dim_product` per the schema above and backfill `product_id` into `shopify_sales`.
3. Query away.

### Working with an AI assistant (e.g. Claude)

**Option A — describe the table.** Give it the table name and column names, then ask your question:

> I'm using DB Browser for SQLite. My table `shopify_sales` has these columns: `product_title, product_vendor, product_type, net_items_sold, gross_sales, discounts, returns, net_sales, taxes, total_sales`. Find the top 10 best-selling products by units sold (`net_items_sold`), showing `product_title`. Give me just the SQLite query.

**Option B — upload the database file** directly and let it inspect the schema itself, then ask your question the same way.

## Tech Stack

- SQLite / SQL
- DB Browser for SQLite (or equivalent client)
- Claude for query drafting and schema review

## Status

Projects 1–2 complete: raw data loaded, schema designed, join reliability characterized. Projects 3–5 (ad attribution, bot/abandonment analysis, dashboard) not yet started in this repo.
