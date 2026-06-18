# QuickBite ETL Pipeline

**Student:** Anush Khachatryan
**Course:** Big Data — Final Exam, Part 2
**Program:** Master's in Data Science in Business

---

## Overview

End-to-end ETL pipeline for QuickBite, a mobile food-delivery platform. The pipeline ingests raw event and operational data from 7 sources, cleans and joins it into an analytics-ready fact table, and materializes daily aggregates and team metrics consumed by 6 business teams.

---

## Architecture

**Bronze → Silver → Gold**

| Layer | What happens |
|---|---|
| **Bronze** | Raw CSVs loaded with explicit schemas, timestamps cast, zero business logic |
| **Silver** | `fact_orders_clean` — deduplicated, status-normalized, LEFT JOINed, anomaly-flagged |
| **Gold** | Daily aggregates + team metrics, BI-ready |

![Architecture Diagram](quickbite_architecture.png)

---

## Data Sources

| Source | Table | Processing mode |
|---|---|---|
| Mobile App Event Stream | `app_events` | Streaming / micro-batch |
| Order/Payment Backend | `orders` | CDC / upsert |
| Payment Gateway | `payments` | Webhook, append-only |
| Restaurant POS | `restaurants` | Batch snapshot, daily SCD |
| Courier App Backend | `couriers` | Batch snapshot, daily SCD |
| Marketing Platform | `marketing_campaigns` | Batch / upsert |
| Promo Engine | `promo_codes` | Hybrid: batch definition + live counter |

---

## Output Tables

| Table | Grain | Key columns |
|---|---|---|
| `fact_orders_clean` | 1 row per order | status_clean, payment_amount, payment_status, campaign_id, has_timestamp_anomaly |
| `agg_restaurant_daily` | restaurant × day | orders, revenue, reject_rate, avg_prep_minutes_actual, rolling_7d_revenue |
| `agg_courier_daily` | courier × day | deliveries, avg_delivery_minutes, on_time_pct |
| `agg_marketing_campaign` | campaign | impressions_proxy, redeemed_orders, conversion_rate, redeemed_revenue |

---

## Data Quality Findings (§3)

11 issues detected and quantified across all 7 tables:

| # | Issue | Rows affected | Fix |
|---|---|---|---|
| 3.1 | Status casing/typo variants (`DELIVERED`, `delivrd`) | 79 (0.53%) | `lower(trim())` + typo map |
| 3.2 | Stale status — `accepted` with `delivered_ts` populated | 20 (0.13%) | Derive status from timestamp trail |
| 3.3 | Orders with no payment record | 220 (1.47%) | LEFT JOIN, not INNER |
| 3.4 | Multiple payments per order — fan-out risk | 30 (0.20%) | Deduplicate with window function |
| 3.5 | `payment.amount` ≠ `orders.total` | 100 (0.81%) | Trust `payment_amount` as source of truth |
| 3.6 | Broken FK: `promo_codes.campaign_id` not in campaigns | 10 (4.76%) | Quarantine, exclude from campaign agg |
| 3.7 | Promo validity window has zero overlap with campaign window | 52 (26%) | **Left unfixed** — requires marketing ops to clarify business rule |
| 3.8 | `current_uses` exceeds `max_uses` | 5 (2.38%) | Cap valid redemptions at `max_uses` |
| 3.9 | Rating present on non-delivered order | 54 (0.75%) | **Left unfixed** — cancelled-with-rating may be legitimate |
| 3.10 | Timestamp ordering violations | 33–94 orders | Flag `has_timestamp_anomaly`, exclude from duration metrics |
| 3.11 | Courier double-booking — overlapping delivery windows | 202 pairs, 151 couriers | Document; filter anomalous rows from time-based metrics |

---

## Team Metrics (§6)

| Team | Metrics |
|---|---|
| **Product / Customers** | Funnel conversion rate (24.94%), repeat order rate (66.77%), time-to-first-order |
| **Restaurants** | Acceptance rate (95.54%), prep-time accuracy (actual vs self-reported), cancellation rate |
| **Couriers** | On-time delivery % (45-min SLA), deliveries per active hour (5.09 avg), avg pickup-to-delivery (24.86 min) |
| **Marketing** | Promo redemption rate (50.18%), incremental revenue per campaign, cost per acquired order ($102.77 avg) |
| **Finance** | Payment failure rate by method (6.4–7.0%), refund $ as % of GMV (11.87%) |
| **Support** | Proxy ticket rate per 1K orders (118.0), refund-driven ticket % (82.37%) |

---

## Automated DQ Checks (§8)

30 programmatic checks across 5 categories — re-runnable on every new data load.

| Category | Checks | Result |
|---|---|---|
| Primary key uniqueness | 7 | 7 PASS |
| Referential integrity | 6 | 5 PASS, 1 FAIL |
| Temporal sanity | 7 | 3 PASS, 4 FAIL |
| Enum validity | 6 | 6 PASS |
| Cross-table reconciliation | 4 | 1 PASS, 3 FAIL |

**21 PASS · 9 FAIL** — all failures match issues documented in §3.

---

## Spark Optimizations (§9)

| Optimization | Details | Impact |
|---|---|---|
| Explicit broadcast hints | `payments_dedup`, `promo_codes`, `restaurants` | Guarantees no shuffle join regardless of table growth |
| Reduced shuffle partitions | 200 → 8 for 14K-row window | Eliminates task scheduling overhead |
| Caching `fact_orders_clean` | Reused 10+ times downstream | 76.5% faster on repeated actions |
| Predicate pushdown | Auto-applied by Catalyst on all broadcast-side scans | Filters pushed to CSV read layer |
| Skew analysis | `city` skew ratio 9.75x (YVN vs DLJ) | Documented; salting demonstrated at 100x scale |

---

## Repository Structure

```
quickbite-etl/
├── quickbite_etl_A_Khachatryan.ipynb   # full solution notebook
├── quickbite_architecture.png           # ETL architecture diagram
├── architecture_diagram.py              # script to regenerate diagram
├── data/
│   ├── app_events.csv
│   ├── orders.csv
│   ├── payments.csv
│   ├── restaurants.csv
│   ├── couriers.csv
│   ├── marketing_campaigns.csv
│   └── promo_codes.csv
└── README.md
```

---

## Requirements

```
pyspark
pandas
matplotlib
plotly
```
