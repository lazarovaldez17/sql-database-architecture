# Project 3 — Following the Money

**Skills demonstrated:** dimensional modeling (lookup/dimension tables), SQL data standardization, marketing analytics (CTR/CPC/ROI), attribution-limitation analysis, data-driven budget reallocation.

This is the third project in a five-part analytics engineering series built on Breaking Games' real Shopify and Meta Ads exports (customer PII synthesized; product, geography, and behavioral data is real — see [`shopify-meta-data/README.md`](shopify-meta-data/README.md)). Where Project 2 modeled *products*, Project 3 models *money*: it takes 422 raw, unlabeled traffic sources and 35 disconnected ad campaigns and turns them into a channel taxonomy, a marketing-metrics framework, and a defendable Q4 ad budget.

---

## 1. Context

Breaking Games spent **$37,200.94** on Meta (Facebook/Instagram) ads across 35 campaigns between January and July 2025. Mike, Head of E-Commerce, has a Q4 budget review with Shari and one question he can't currently answer:

> *"Did the ads work? Where should the next dollar go?"*

Nobody could answer that going in, because ad spend and site revenue live in two systems with no shared key — Meta reports campaign performance, Shopify reports orders, and nothing joins them. This project builds the missing layer between the two: a clean traffic taxonomy, a metrics framework for judging campaigns on their own terms, and a budget plan that reallocates the freed money rather than asking for more.

## 2. Objectives

1. Sort 422 raw referrer strings into a small, decision-ready set of channels.
2. Learn attribution well enough to know which claims the data *can* and *can't* support, and pick the honest approach for this dataset.
3. Learn the three core paid-marketing metrics (CTR, CPC, ROI) and use them to rank all 7 campaigns that actually spent money.
4. Ship a Q4 ad allocation plan: what to keep, what to cut, and where the freed budget goes — each pick backed by one number and one sentence.

## 3. Problems & Solutions

| Problem | Why it's hard | Solution |
|---|---|---|
| **422 distinct referrer strings, no channel column.** `google` (organic search) and `googlesyndication` (Google's paid ad network) share a prefix but are opposite in intent; `facebook`, `instagram`, and `meta` are three names for one channel. | A naive `GROUP BY referrer` either fragments obviously-related traffic or merges paid and free traffic into the same bucket Mike is making a budget call on. | Designed and built `dim_referrer`, a lookup table mapping every referrer to a `channel` and a `channel_type` (Paid / Organic / Owned), validated against real traffic share before trusting it (see §5). |
| **No attribution path.** No UTM tags, no session-to-purchase link, and `referrals` rows are aggregated by referrer + landing page — not per visitor. | You cannot say *"this Facebook ad caused this sale."* Any tool that claims to would be reporting a guess as a fact. | Used **directional correlation** instead of a formal attribution model (last-touch, first-touch, linear, etc.), computing cost-per-pageview and cost-per-checkout-session per channel, and stating the limitation explicitly rather than hiding it in a footnote. |
| **`results` means four different things across campaigns.** Landing-page views, add-to-carts, purchases, and reach are all stored in the same `results` / `cost_per_result` columns. | Reading `cost_per_result` alone, $0.22 looks "cheaper" than $70.76 — but one is a page view and the other is a purchase. Ranking them together is a category error. | Always read `result_indicator` first and grouped campaigns by objective before ranking. Of 35 campaigns, only 6 ever recorded a result; 28 never spent a dollar; the 35th spent $12,917.08 (the largest campaign in the account) and recorded nothing — see §6. |
| **`meta_campaigns` and `meta_ad_sets` share no foreign key.** Ad sets (audience-targeting detail) can't be reliably joined back to their parent campaign. | Naming conventions hint at relationships, but nothing in the schema confirms them. | Kept campaign-level and ad-set-level analysis separate rather than inferring a join the data doesn't support — a smaller, defensible claim beats a larger, unverifiable one. |

## 4. Design Decisions

**`dim_referrer` as a dimension table, not a CASE statement.** Channel logic lives in one lookup table (`referrer TEXT PRIMARY KEY, channel TEXT, channel_type TEXT`) that every query joins against, instead of being copy-pasted into every query as a `CASE WHEN referrer IN (...)`. One place to fix a miscategorized referrer; every downstream query inherits the fix automatically.

**Six channels, not more.** `Paid Social`, `Organic Search`, `Paid Search`, `Organic Social`, `Owned`, `Other`. Enough granularity to separate the channel Mike pays for from the channels he doesn't, without fragmenting the long tail (~388 of the 422 referrers) into buckets too small to act on.

**`channel_type` uses three labels: Paid, Organic, Owned.** This is the field Mike actually budgets against. Collapsing six channels into three types makes the "where does the money go" question answerable in one `GROUP BY`.

**Paid-only metrics are `NULL`, not `0`, on non-paid channels.** `cost_per_pageview` and `cost_per_checkout_session` only apply to Paid Social, since Meta spend is the only spend data available. Writing `$0` for Organic Search would claim "this channel is free and infinitely efficient" — a data quality decision, not just a formatting one.

**Normalized blank referrers to `'direct'` at the query level** (`COALESCE(NULLIF(referrer, ''), 'direct')`) rather than leaving them as empty strings, so `dim_referrer` has one canonical key instead of two representations of the same thing.

## 5. E-Commerce Analytics

### Channel breakdown (Jan–Jul 2025)

| Channel | Type | Share of pageviews | Checkout conv. rate |
|---|---|---|---|
| Owned (direct + email) | Owned | 46.7% | 0.237% |
| Paid Social (Meta) | Paid | 27.3% | 0.124% |
| Organic Search | Organic | 24.2% | **0.476%** |
| Other (long tail, ~388 referrers) | Other | 1.4% | 0.374% |
| Organic Social | Organic | 0.3% | 0.216% |
| Paid Search (non-Meta) | Paid | 0.3% | 0.231% |

*Percentages rounded to one decimal place; column sums to 100.2% due to rounding, not double-counted traffic.*

**Key finding:** the channel receiving 100% of the ad budget converts *worse* than free organic search traffic — Paid Social converts at 0.124% versus Organic Search's 0.476%, roughly 4x lower. Paid Social spend also works out to **$0.41 per pageview and $329 per checkout session**, against a typical order value in the same window of roughly $58–$81. That gap is the analytical backbone of the Q4 plan below.

### Campaign metrics framework

Three metrics, three questions:

| Metric | Formula | Answers |
|---|---|---|
| **CTR** | `link_clicks / impressions` | Is the creative compelling? |
| **CPC** | `spend_usd / link_clicks` | What does each visitor cost? |
| **ROI / ROAS** | `(revenue − spend) / spend` | Did the money come back? (Directional only — see §3.) |

Of 35 campaigns, only **7 ever spent money** — those are the ones ranked. Names below are kept exactly as exported from Meta Ads Manager (`DA` = the agency/account's internal campaign prefix, `TOF` = top of funnel, `ATC` = add to cart, `Advantage Plus` = Meta's automated campaign product, `MAT Test` = a materials/creative test) rather than renamed, since a hiring reader should see what a real ad-platform export actually looks like:

| Campaign | Status | Spend | Result type | Cost/result | CTR |
|---|---|---|---|---|---|
| DA \| Sales | active | $12,917.08 | *none recorded* | — | 0.84% |
| *DA \| Traffic TOF | active | $8,634.89 | landing page view | $0.22 | 1.40% |
| *DA \| Sales \| Advantage Plus | active | $6,863.81 | purchase | $70.76 | 1.00% |
| DA \| MAT Test \| Dwellings \| Traffic-LPV | inactive | $4,344.57 | landing page view | $0.24 | 1.75% |
| DA \| Reach \| Retargeting | active | $1,612.34 | reach (per 1k) | $1.33 | 0.10% |
| DA \| Sales \| ATC | inactive | $1,441.94 | add to cart | $4.96 | 2.51% |
| DA \| MAT Test \| We're Doomed \| Traffic-LPV | inactive | $1,386.31 | landing page view | $0.39 | 0.78% |

## 6. Q4 Ad Allocation Plan


**The core finding:** $12,917.08 — over a third of the entire Meta budget — went to a single campaign ("DA | Sales") with *zero* recorded results of any kind. That's the headline, not a footnote.

| Decision | Campaign | Backed by |
|---|---|---|
| Keep | *DA \| Traffic TOF | $0.22/landing-page view — cheapest result in the account |
| Keep | DA \| Reach \| Retargeting | $1.33 per 1,000 reached, 1.2M people, 4% of spend |
| Cut | DA \| Sales | $12,917.08 spent, zero results recorded |
| Cut in half | *DA \| Sales \| Advantage Plus | $70.76/purchase — at or above typical order value, unclear margin |
| New | Restart DA \| Sales \| ATC | Last run: $4.96/add-to-cart, 2.51% CTR (best in account), currently paused |

**The money math:** cutting DA | Sales fully and Advantage Plus by half frees **$16,348.99** — no new budget requested. That figure funds restarting the ATC campaign at meaningfully greater scale, with room left over to reinforce both Keep picks.

---


## Limitations, stated plainly

- **Correlation, not attribution.** No UTM tags and no session-to-purchase link exist in this dataset. Every channel-to-revenue statement here is "this much traffic happened alongside this much revenue," never "this traffic caused this revenue."
- **`cost_per_result` is only comparable within the same `result_indicator`.** A $0.22 landing-page view and a $70.76 purchase are not on the same scale — the campaign table above is grouped for that reason.
- **The ~$58–$81 order-value estimate is a partial-period sample** (`shopify_orders` covers July 2025 only, one month of the seven-month ad window), not a full-period figure — treated as directional, not exact.
