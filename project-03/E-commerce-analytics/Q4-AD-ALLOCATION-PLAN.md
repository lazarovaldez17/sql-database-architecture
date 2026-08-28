# Breaking Games — Q4 Ad Allocation Plan

**Prepared for:** Mike, Head of E-Commerce · **Decision:** where the Q4 Meta budget goes · **Budget in question:** ~$37,200.94 (Jan–Jul 2025 Meta spend, the baseline being reallocated for Q4)


**The headline:** $12,917.08 — 34.7% of the entire Meta budget — went to one campaign ("DA | Sales") with zero recorded results of any kind. Not a low result. No result field at all.

---

## Keep running

### *DA | Traffic TOF

- **Decision:** Keep, and consider scaling further.
- **Backed by:** $0.22 per landing-page view on 39,419 views from $8,634.89 spend — the cheapest result of any campaign in the account. CTR 1.40%, CPC $0.21.
- **Expected:** Keeps the top of funnel full at the account's best price. Additional spend here should buy landing-page views at roughly the same rate.
- **Caveat:** A landing-page view isn't a sale. There's no link from this campaign's clicks to a purchase, so we can't say how many of these visitors ever buy.

### DA | Reach | Retargeting

- **Decision:** Keep.
- **Backed by:** $1,612.34 spend reached 1,208,484 people (past visitors, excluding recent purchasers) at $1.33 per 1,000 reached — the cheapest reach in the account, for just 4% of total spend.
- **Expected:** Keeps warm, already-engaged visitors thinking about Breaking Games heading into Q4 holiday shopping, for minimal budget.
- **Caveat:** Reach isn't clicks or sales. Its CTR (0.10%) is far below every other campaign — expected for a reach objective, not a click objective, but worth watching if it doesn't hold.

## Cut or reduce

### DA | Sales

- **Decision:** Cut.
- **Backed by:** $12,917.08 spent — over a third of the entire Meta budget — with no `result_indicator` and no results recorded at all. It's the only spending campaign in the account with literally nothing to measure.
- **Expected:** Frees $12,917.08. Since there's no data showing this campaign produced anything, nothing measurable is lost by cutting it.
- **Caveat:** No recorded result doesn't prove zero impact — Meta simply isn't tracking anything for this campaign. But "we can't tell" isn't a defensible reason to keep funding a third of the budget.

### *DA | Sales | Advantage Plus Campaign

- **Decision:** Cut in half.
- **Backed by:** $70.76 per purchase on 97 purchases from $6,863.81 spend — at or above the ~$58 average order value seen in July 2025 (the one month of order-level detail available inside this ad-spend window), before the cost of the product itself.
- **Expected:** Trimming to ~$3,431.91 protects the only campaign in the account with real, attributed purchase data, while capping downside if the margin doesn't clear the $70.76 hurdle.
- **Caveat:** The $58 AOV is a one-month sample, not the full 7-month window, and there's no product cost/margin data — we can't say whether $70.76/purchase is a profit or a loss. This is a risk-management trim, not a proven-loser cut.

## New investment

### Restart DA | Sales | ATC

- **Decision:** New (restart a paused campaign).
- **Backed by:** Its last run: $4.96 per add-to-cart on 291 ATCs from $1,441.94 spend, with a 2.51% CTR — the strongest creative engagement of any campaign that ever spent money on this account. It's currently paused.
- **Expected:** The $16,348.99 freed by the two cuts funds restarting this at several times its old scale, with room left to add to the two Keep campaigns above.
- **Caveat:** This isn't truly new — it's a proven campaign, not a proven-for-Q4 one. Its numbers are from earlier in the year; watch cost-per-result closely for the first two weeks before committing the full amount.

---

## The money math

Cutting DA | Sales ($12,917.08, zero recorded results) and trimming Advantage Plus by half ($3,431.91, thin-to-unclear margin) frees **$16,348.99** — no new budget required.

```
$12,917.08  (DA | Sales, full cut)
+ $3,431.91  (Advantage Plus, half cut)
-----------
$16,348.99  freed → funds restarting DA | Sales | ATC, with room to reinforce the Keep picks
```

## Supporting data

Every campaign that spent money on Meta, Jan–Jul 2025 (28 of 35 campaigns never spent a dollar and are excluded):

| Campaign | Status | Spend | Result type | Results | Cost/result | CTR |
|---|---|---|---|---|---|---|
| DA \| Sales | active | $12,917.08 | — | — | — | 0.84% |
| *DA \| Traffic TOF | active | $8,634.89 | landing page view | 39,419 | $0.22 | 1.40% |
| *DA \| Sales \| Advantage Plus | active | $6,863.81 | purchase | 97 | $70.76 | 1.00% |
| DA \| MAT Test \| Dwellings \| Traffic-LPV | inactive | $4,344.57 | landing page view | 17,753 | $0.24 | 1.75% |
| DA \| Reach \| Retargeting | active | $1,612.34 | reach (per 1k) | 1,208,484 | $1.33 | 0.10% |
| DA \| Sales \| ATC | inactive | $1,441.94 | add to cart | 291 | $4.96 | 2.51% |
| DA \| MAT Test \| We're Doomed \| Traffic-LPV | inactive | $1,386.31 | landing page view | 3,540 | $0.39 | 0.78% |

**Source:** `breaking_games.db` (`meta_campaigns`, `referrals`, `dim_referrer`, `shopify_orders`/`shopify_sales`). Cost-per-result is only comparable within the same result type — a $0.22 landing-page view and a $70.76 purchase are not on the same scale. No UTM tags or session-to-purchase link exist in this data, so campaign-to-revenue is correlation, not true attribution.
