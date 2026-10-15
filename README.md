# Skincare Demand Intelligence

**Is a 10,000-product skincare assortment being managed where customer demand actually is?**

A customer-engagement analysis of a synthetic Sephora-style skincare catalog (10,000 products, 27 attributes). The project segments products by customer demand and satisfaction, then tests whether inventory and discounting decisions line up with them.

> **Data note:** the dataset is synthetic and product-level (no transactions or customer IDs). Customer behavior is inferred from engagement signals: `loves_count` (wishlist intent), `review_count` (post-purchase engagement), `rating` and `recommend_rate` (satisfaction). See *Limitations*.

## Business problem
Merchandising teams must decide where to put scarce inventory, promotion budget and quality attention. I answer four questions:

1. How concentrated is customer demand?
2. Which products are stars, hype traps or hidden gems?
3. Are stock levels and discounts aligned with demand and satisfaction?
4. Do product attributes (claims, price tier, category, brand) explain engagement?

## Key findings
| # | Finding | Evidence |
|---|---|---|
| 1 | **Demand is extremely concentrated** | Top 10% of products = **57%** of all loves; top 1% = **21%**; bottom 50% = **9%** |
| 2 | **Half of high-engagement products have weak satisfaction** | 1,187 "hype traps" (median 25K loves, 69% recommend) vs 1,225 "stars" (23K loves, 82% recommend) |
| 3 | **3,601 "hidden gems" are as well liked as stars but get ~1/7 of the engagement** | Median 3.4K vs 23.1K loves; recommend rate 86% vs 82% |
| 4 | **Stock isn't prioritised by demand** | 14.7% of the top-demand decile is low/out of stock vs 14.2% elsewhere (p = 0.69); 147 products, 68 fully out of stock, holding 13.3% of top-decile engagement |
| 5 | **Discounts are untargeted** | On-sale rate is 11.7% for stars, 9.8% for hype traps, 10.4% for hidden gems (p = 0.48) |
| 6 | **Attributes don't explain engagement** | 0 of 8 claims/flags significant after Bonferroni correction; category, brand, concern and price tier also non-significant |

## Recommendations
1. **Protect the head of the curve:** demand-weighted safety stock; replenish the 147 at-risk top-decile products first (`outputs/action_list_high_demand_stock_risk.csv`).
2. **Audit hype traps** before spending more on them: check returns and review text for expectation gaps.
3. **Promote hidden gems** (sampling, placement, bundles) and measure lift with a holdout group.
4. **Retarget discounts** away from stars towards hidden gems and new launches.
5. **Decide at product level**, not by claim, price band or category.

## Methods
- Data quality checks (nulls, duplicates, internal consistency, plausibility of rating vs recommend-rate)
- Bayesian-shrunk rating so low-review products can't outrank well-reviewed ones on noise
- Satisfaction index = mean percentile rank of adjusted rating and recommend rate
- 2x2 segmentation: top-quartile engagement x median satisfaction
- Concentration (Lorenz-style) analysis; chi-square and Spearman tests for stock/discount alignment
- Bootstrapped median ratios (95% CI) with Mann-Whitney tests and Bonferroni correction for attribute effects

## Repository layout
```
data/                 sephora_skincare_dataset.csv
notebooks/            skincare_demand_analysis.ipynb   (executed, narrative + code)
outputs/
  figures/            4 charts (PNG)
  product_segments.csv                         every product with its segment
  action_list_high_demand_stock_risk.csv       prioritised replenishment list
  key_metrics.json                             every number quoted above
  skincare_demand_analysis.html                read-only export of the notebook
```

## Reproduce
```bash
pip install -r requirements.txt
cd notebooks && jupyter nbconvert --to notebook --execute --inplace skincare_demand_analysis.ipynb
```

## Limitations
- **Synthetic, product-level data.** This is engagement analysis, not purchase-behavior analysis; loves and reviews are proxies.
- **Flat relationships are probably a data-generation artefact.** Rating and recommend rate are uncorrelated (Spearman -0.01) and rating noise doesn't shrink with review volume, which real data would show. I report null results as null rather than forcing a story.
- **Segment thresholds are judgement calls** (top quartile / median); sensitivity is worth checking.
- **No revenue or margin data**, so opportunity is sized in share of engagement, not dollars.

**With real data I would add:** RFM and cohort retention, repeat-purchase and basket analysis, promo lift via holdout tests, and a returns-rate join to validate the hype-trap hypothesis.
