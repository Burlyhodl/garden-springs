# Shopify + GA4 + Google Ads + Meta Ads + Google Search Console Dashboard Blueprint

This blueprint describes how to build a Looker Studio dashboard that blends Shopify, GA4, Google Ads, Meta Ads, and Google Search Console while keeping purchase-centric KPIs accurate and transparent.

## Data integrations
- **Shopify**: Orders, transactions, and line items with `order_id`, `transaction_id`, `sku`, `item_name`, `quantity`, `price`, `discount`, `shipping`, `tax`, `created_at`, and any available `source/medium`.
- **GA4**: `event_name = purchase` events with `transaction_id`, `session_source`, `session_medium`, `source`, `medium`, `campaign`, `date`, `event_value`, and `items_purchased`.
- **Google Ads & Meta Ads**: Cost, clicks, impressions, and purchase-only conversions with `date`, `campaign`, `ad_group/adset`, `ad_id`, `source` (`google` or `facebook`), and `medium` (`cpc`/paid).
- **Google Search Console**: Queries and landing pages with `date`, `page`, `clicks`, `impressions`, `CTR`, and `position`.

## Blending model
- **Primary join**: `date` + normalized `source/medium` for spend vs. conversions.
- **Purchase alignment**: `transaction_id` (GA4) ↔ `order_id` (Shopify); fall back to `date` + `source/medium` for aggregated comparisons.
- **Channel normalization field**: create a calculated dimension (e.g., `Paid Source`) that buckets traffic into `Google Ads`, `Meta Ads`, `Other Paid`, `Organic`, `Direct`, `Referral`.
- Prefer **left joins** anchored on Shopify orders when reconciling with GA4 and ad platforms.

## KPI dictionary (purchase-only)
Create a dedicated tab describing each field’s source, formula, currency, and aggregation:
- Amount spent (ad cost)
- Orders (Shopify)
- Transactions (GA4 purchases)
- Items purchased
- Purchase revenue (GA4 `event_value`)
- Order revenue (Shopify gross)
- Transaction revenue (GA4)
- Conversions (limited to purchase events)
- Derived: ROAS, CPA, match-rate %, revenue variance

## Metric validation & deduplication
- Flag mismatches between Shopify orders and GA4 transactions by both `transaction_id` and `date/source`.
- Data Quality scorecards:
  - `GA4_transactions_minus_Shopify_orders`
  - `% GA4 transactions matched to Shopify orders`
  - `% revenue difference (GA4 vs Shopify)`
- Explicitly label overlapping metrics (GA4 purchases vs Shopify orders, GA4 purchase revenue vs Shopify gross, platform conversions vs GA4/Shopify) and apply warnings when variance > 5%.
- Detect duplicate GA4 transactions: flag when `count_distinct(transaction_id) < count(transaction_id)` per `date/source`.

## Segmentation
Break down KPIs by:
- Platform: Google Ads, Meta Ads
- Channel/Medium: paid search, paid social, organic search (GSC), direct, referral
- Campaign/adset/ad group for paid platforms; landing page for GSC/organic
- Optional: device category and geo (country/region) when available

## Suggested pages & visuals
1. **Executive Summary**
   - Scorecards: spend, orders, transactions, items, revenue (Shopify vs GA4 side-by-side).
   - Blended ROAS and CPA per platform; date range and platform selectors.
2. **Channel Performance**
   - Stacked bar or heatmap: spend, orders, revenue by platform and medium.
   - Dual-axis time series: spend vs purchases with anomaly bands.
3. **Attribution & Source Detail**
   - Blended table: ad spend with GA4 purchases by `source/medium`, highlighting mismatches.
   - Optional: GA4 data-driven attribution vs platform-reported conversions side-by-side.
4. **Product & Landing Insights**
   - Top products (Shopify) with revenue and quantity; overlay GA4 item views to reveal funnel gaps.
   - GSC landing pages with clicks/impressions/CTR/position blended with GA4 purchase count.
5. **Data Quality Panel**
   - Match-rate visuals, duplicated transaction detector, revenue variance bars, and a Known Issues note.

## Best practices
- Do not sum platform-reported conversions with GA4/Shopify; present them side-by-side.
- Keep currency and timezone consistent across connectors.
- Use filters only on dimensions present in all blends (date, source/medium) to avoid aggregation errors.
- GA4: filter to `event_name = purchase`; Ads: include only purchase-mapped conversions.
- Validate totals monthly by exporting source summaries and reconciling with dashboard aggregates.

## Known issues to document
- GA4 vs Shopify tracking gaps (ad blockers, server-side lag)
- Platform conversion windows vs GA4 same-day attribution
- Currency rounding and tax/shipping inclusion differences
- GA4 sampling/thresholding or connector limits

## Implementation checklist
- Normalize `source/medium` into standard buckets before blending.
- Build calculated fields: ROAS, CPA, match-rate %, revenue variance, duplicate transaction flag.
- Add parameter controls for date range and platform selection.
- Verify data joins starting at the transaction_id level before enabling aggregate views.
