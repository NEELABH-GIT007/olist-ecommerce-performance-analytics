# Brazilian E-Commerce Performance & Customer Analytics

**SQL | DuckDB | Python | Pandas | Tableau | Business Intelligence**

> **Status:** Active development. Core data-quality work, analytical modeling, SQL analysis, Python EDA, reporting-layer design, and the Executive Overview analytical build are complete. Product & Category dashboard development is in progress.

## Project overview

This project analyzes the Brazilian Olist e-commerce dataset to understand commercial performance, customer behavior, product/category performance, seller activity, payments, geography, delivery performance, and customer experience.

The project is intentionally built as more than a dashboard. It starts with business requirements and source validation, then progresses through data cleaning, grain-aware analytical modeling, SQL analysis, Python exploratory analysis, a reusable reporting layer, and Tableau dashboards.

## Business problem

The source data is distributed across orders, customers, order items, products, sellers, payments, reviews, category translation, and geolocation tables. Several relationships are one-to-many, so directly flattening all source tables would duplicate measures and produce incorrect KPIs.

The project therefore focuses on two goals:

1. Produce reliable business insights from the marketplace data.
2. Build reusable analytical models whose grain and metric behavior are explicit and validated before visualization.

## Validated project scale

| Metric | Result |
|---|---:|
| Orders | 99,441 |
| Order items | 112,650 |
| Merchandise value | R$13,591,643.70 |
| Freight value | R$2,251,909.54 |
| Payment value | R$16,008,872.12 |
| Unique customers | 96,096 |
| Repeat-customer rate | 3.12% |
| On-time delivery rate | 93.23% |
| Average review score | 4.09 |

## Data architecture

```text
Raw Olist data
      ↓
Data profiling & relationship validation
      ↓
clean.*
      ↓
analytics.*
      ↓
reporting.*
      ↓
Tableau dashboards
```

### Core analytical objects

| Object | Grain | Primary use |
|---|---|---|
| `analytics.order_analysis_base` | One row per order | Executive KPIs, customers, delivery, reviews |
| `analytics.item_analysis_base` | One row per order item | Product, category and seller commercial analysis |
| `reporting.order_category_base` | One row per order-category | Category commercial and review analysis |
| `reporting.order_seller_base` | One row per order-seller | Seller commercial and experience analysis |
| `reporting.payment_analysis_base` | One row per payment record | Payment type and installment analysis |
| `reporting.state_marketplace_summary` | One row per state | Customer-side vs seller-side geography and delivery |

This structure avoids a universal flat table because different business questions require different valid grains.

## Selected findings

- **Repeat purchasing is limited:** 2,997 of 96,096 unique customers were repeat customers, a **3.12% repeat-customer rate**.
- **Delivery performance is associated with customer experience:** reviewed on-time orders averaged **4.29**, compared with **2.27** for late deliveries.
- **Category value is concentrated:** the **top 10 categories generated 62.36%** of merchandise value.
- **Customer demand is geographically concentrated:** São Paulo represented approximately **41.98% of customer orders**.
- **Customer and seller geography are not balanced:** São Paulo represented approximately **38.28% of customer merchandise value** but **64.40% of seller merchandise value**.
- **Volume and value are not interchangeable:** categories with similar merchandise value can have materially different item volumes and average item values.

These are descriptive findings from the observed data. Association is not treated as causation.

## Current Tableau progress

### Dashboard 1 — Executive Overview
Functional analytical build completed:
- KPI strip
- Monthly orders and merchandise value
- Top categories by merchandise value
- Review score by delivery outcome
- Shared reporting-period controls across different data grains

### Dashboard 2 — Product & Category Performance
In progress:
- `PC_01_Category_Ranking` — complete
- `PC_02_Category_Value_vs_Volume` — complete
- `PC_03_Category_Review_Performance` — in progress

Planned later dashboards:
- Customer & Geography
- Seller Performance
- Delivery & Customer Experience
- Payment Analysis

## Technology

- **SQL / DuckDB:** transformation logic, analytical models, joins, aggregations, CTEs, window functions, validation and reconciliation
- **Python / Pandas / NumPy:** exploratory analysis and reproducible investigation
- **Matplotlib:** analytical visualizations during EDA
- **Tableau:** KPI reporting, interactive analysis and dashboard delivery
- **Git / GitHub:** version control, documentation and portfolio presentation

## Analytical approach

```text
Business requirement
      ↓
Source and grain validation
      ↓
Data-quality evidence
      ↓
Transformation rule
      ↓
SQL implementation
      ↓
Validation / reconciliation
      ↓
Business analysis
      ↓
Python EDA where it adds new evidence
      ↓
Reusable reporting model
      ↓
Tableau
```

Key principles:
- Define grain before joining data.
- Preserve raw data and document transformations.
- Do not silently convert missing observations to zero.
- Separate additive and non-additive measures.
- Reconcile new transformations to validated control totals.
- Avoid redundant tests and unnecessary computation.
- Use association language unless causal evidence exists.
- Avoid misleading comparisons across incomplete reporting periods.

## Repository structure

```text
.
├── README.md
├── .gitignore
├── docs/
│   ├── README.md
│   ├── architecture.md
│   └── ai_usage.md
├── sql/
│   └── README.md
├── notebooks/
│   └── README.md
├── tableau/
│   └── README.md
└── assets/
    └── README.md
```

The repository is being populated incrementally from the validated local project. Raw source data, local DuckDB files, virtual environments, and machine-specific files are intentionally excluded.

## Data limitations

- Merchandise value represents product value, **not revenue or profit**.
- Profitability, margin, discount elasticity, inventory optimization, formal forecasting and causal inference are out of scope with the available source data.
- Boundary-year coverage is incomplete, so annual totals should not be interpreted as like-for-like full-year YoY performance.
- Missing reviews mean no observed review, not a zero review score.
- State-level distinct customer counts are not additive because some customers appear in more than one state.
- Extreme observations are not automatically treated as invalid.

## Use of Generative AI

Generative AI was used as a **learning and productivity assistant** for technical explanation, design review, debugging support, documentation, and challenging analytical assumptions.

AI output was not treated as ground truth. SQL logic, grains, transformation rules, KPI definitions, reconciliation results, Tableau calculations, and business interpretations were independently checked against the underlying data before being accepted.

## Roadmap

- Complete Product & Category Performance dashboard
- Build Customer & Geography dashboard
- Build Seller Performance dashboard
- Build Delivery & Customer Experience dashboard
- Build Payment Analysis dashboard
- Complete final Tableau UI/UX pass
- Add project screenshots and architecture visuals
- Add validated SQL and notebook artifacts to this repository
- Publish final project case study
