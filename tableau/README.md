# Tableau Reporting & Dashboard Documentation

## 1. Overview

Tableau is the interactive business intelligence and reporting layer of the Brazilian E-Commerce Performance & Customer Analytics project.

The purpose of this layer is to translate validated analytical outputs into interactive dashboards that help stakeholders understand:

- Marketplace commercial performance
- Product and category contribution
- Customer purchasing behaviour
- Geographic demand distribution
- Seller performance and concentration
- Delivery outcomes and customer experience
- Payment-method composition

The Tableau implementation follows a grain-aware reporting architecture. Rather than combining all source tables into one large flat dataset, dashboards use analytical and reporting models designed for their specific business questions.

All four dashboards have been built, visually refined and functionally checked.

## 2. Reporting Architecture

The Tableau layer is the final stage of the project's analytical architecture.

```text
Raw Olist CSV Data
        |
        v
Data Quality & Relationship Validation
        |
        v
Clean Layer
(clean)
        |
        v
Analytical Models
(analytics)
        |
        v
Reporting Models
(reporting)
        |
        v
Tableau Data Sources
        |
        v
Interactive Dashboards
D01 | D02 | D03 | D04
```

### Data architecture principles

- Raw source data is preserved separately from cleaned and transformed data.
- Cleaning rules are documented and validated before analytical use.
- Analytical models have explicit and documented grains.
- One-to-many relationships are handled through appropriate aggregation before measures are combined.
- Reporting models are designed around specific analytical requirements.
- Tableau consumes reusable analytical and reporting outputs instead of repeatedly rebuilding business logic inside individual worksheets.
- KPI calculations are reconciled against validated SQL results.

## 3. Tableau Data Sources and Model Grains

The reporting layer contains four reusable reporting models, alongside two analytical models.

| Tableau data source / model | Grain | Primary purpose |
|---|---|---|
| `analytics.order_analysis_base` | One row per order | Executive, customer and delivery analysis |
| `analytics.item_analysis_base` | One row per order item | Product-level analysis |
| `reporting.order_category_base` | One row per order and analytical category | Category performance and category-level customer experience |
| `reporting.order_seller_base` | One row per order and seller | Seller performance and seller concentration |
| `reporting.payment_analysis_base` | One row per order and payment sequence | Payment-method analysis |
| `reporting.state_marketplace_summary` | One row per state | Customer-side and seller-side geographic comparisons |

### Important grain considerations

**Order-level model**

`analytics.order_analysis_base` contains one record per order. Order-level measures can be aggregated directly at this grain without unnecessary `DISTINCT` operations.

**Item-level model**

`analytics.item_analysis_base` contains one record per order item. Product-level measures are calculated at item grain. Order counts require distinct order identification when aggregating multiple items.

**Order-category model**

`reporting.order_category_base` aggregates item records to order-category grain before attaching order context. This prevents item-level multiplication of order-level measures.

**Order-seller model**

`reporting.order_seller_base` aggregates item records to order-seller grain. Seller measures are analyzed at the seller and order-seller levels.

**Payment-record model**

`reporting.payment_analysis_base` retains payment-record grain. Payment values are additive across payment records, but distinct order counts across payment methods are not additive.

**State summary model**

`reporting.state_marketplace_summary` independently aggregates customer-side and seller-side activity before combining the results at state grain. This prevents incorrect joins between incompatible underlying populations.

## 4. Dashboard Documentation

### D01 — Executive Overview

**Business purpose**

Provide a high-level view of marketplace commercial performance, customer activity and delivery experience.

**Primary data source:** `analytics.order_analysis_base`

**Worksheets and components**

- Executive KPI strip
- Monthly Orders and Merchandise Value
- Top Categories by Merchandise Value
- Customer Review Score by Delivery Outcome

**Key metrics**

- Total Orders
- Merchandise Value
- Average Order Value
- Unique Customers
- Average Review Score
- On-Time Delivery Rate

**Analytical focus**

The dashboard provides an overview of transaction activity, commercial value, category contribution, monthly trends and the relationship between delivery outcome and customer reviews.

### D02 — Product & Category Performance

**Business purpose**

Evaluate product-category contribution through merchandise value, item volume, customer experience and product-level performance.

**Primary data sources:**
- `reporting.order_category_base`
- `analytics.item_analysis_base`

**Worksheets and components**

- Selected Category KPI Snapshot
- Category Ranking
- Category Value vs. Volume
- Category Review Performance
- Top Products in Selected Category

**Interactive behaviour**

Selecting a category updates the selected-category KPI snapshot and the Top Products view.

The selected-category interaction uses a Tableau parameter action. The underlying category value remains available for calculations while readable category aliases are used for presentation.

**Analytical focus**

The dashboard helps identify categories with high merchandise value, categories with high item volume, differences between commercial contribution and customer-review performance, and leading products within the selected category.

### D03 — Customer & Geographic Performance

**Business purpose**

Understand customer purchasing frequency, repeat-customer behaviour, customer value and geographic distribution.

**Primary data source:** `analytics.order_analysis_base`

**Worksheets and components**

- Customer KPI Snapshot
- Customer Order Frequency
- Merchandise Value by Customer Type
- Average Merchandise Value per Customer
- Customer Orders by State
- Repeat Customer Rate by State

**Analytical focus**

The dashboard compares one-time and repeat customers, examines their merchandise-value contribution, and evaluates geographic variation in customer activity and repeat purchasing.

**Important interpretation**

State-level distinct customer counts represent customer presences within individual states. They are not additive across states because a customer can be associated with more than one state during the observed period.

### D04 — Marketplace Operations & Customer Experience

**Business purpose**

Examine seller commercial contribution, seller concentration, delivery-related review outcomes and payment-method composition.

**Primary data sources:**
- `reporting.order_seller_base`
- `analytics.order_analysis_base`
- `reporting.payment_analysis_base`

**Worksheets and components**

- Seller Performance
- Seller Concentration
- Delivery Outcome vs. Customer Review Score
- Payment Methods

**Analytical focus**

The dashboard examines the relationship between seller item volume and merchandise value, identifies the contribution of leading sellers, compares review scores between delivery outcomes, and analyses payment value by payment method.

**Important interpretation**

Payment records can contain multiple records for the same order. Method-level distinct order counts can therefore overlap and must not be summed to obtain the overall distinct order count.

## 5. Reporting-Period Controls and Interactions

All four dashboards include reporting-period controls using shared Start Date and End Date parameters.

### Date-filter implementation

The reporting-period logic is based on the order purchase date.

Conceptually:

```tableau
DATE([order_purchase_timestamp]) >= [p_Start Date]
AND
DATE([order_purchase_timestamp]) <= [p_End Date]
```

The calculated filter is applied to the relevant worksheets and data sources.

For Top N worksheets and calculations that depend on the filtered population, the reporting-period filter is placed in context where required.

### Dashboard-specific interaction

| Dashboard | Interaction |
|---|---|
| D01 | Reporting-period controls |
| D02 | Reporting-period controls and selected-category parameter action |
| D03 | Reporting-period controls, including period-aware customer frequency and repeat-customer calculations |
| D04 | Reporting-period controls across seller, delivery and payment analyses |

### Period-aware customer calculations

Customer frequency and customer type in D03 are evaluated within the selected reporting period.

The customer order count is calculated at customer grain using the orders included in that period. This ensures that a customer is not classified as repeat solely because of orders outside the selected reporting window.

## 6. KPI Definitions and Calculation Rules

| KPI | Definition | Important consideration |
|---|---|---|
| Total Orders | Count of order records at order grain | Direct row count from the one-row-per-order model |
| Total Items | Count of order-item records | Item-level measure |
| Merchandise Value | Sum of item price values | Not equivalent to profit or automatically equivalent to company revenue |
| Average Order Value | Merchandise Value divided by Total Orders | Uses the full order population, including orders without observed item value, under the documented reporting definition |
| Unique Customers | Distinct customer unique IDs | Customer identity is based on `customer_unique_id` |
| Repeat Customer Rate | Customers with more than one order divided by unique customers in the selected period | Period-aware customer classification |
| Average Review Score | Average of the order-level aggregated review score | Order-weighted average, not raw-review-record-weighted average |
| On-Time Delivery Rate | Eligible delivered orders with non-positive delivery variance divided by eligible delivered orders | Orders without valid delivery variance are excluded |
| Payment Value | Sum of payment values | Additive at payment-record grain |
| Payment Value Share | Payment-method value divided by total payment value in the selected reporting period | Table calculation across payment methods |

### Additional calculation safeguards

- Null review scores are excluded from review averages.
- Missing review records are not interpreted as zero-star reviews.
- Delivery eligibility is restricted to delivered orders with valid delivery variance.
- Customer and order denominators are selected according to the analytical question.
- Measures from different grains are not combined without appropriate aggregation.
- Payment-method distinct order counts are not summed to derive overall distinct orders.

## 7. Validation and Reconciliation

The Tableau reporting layer was validated against the SQL analytical and reporting outputs.

### Core validation results

| Validation item | Expected result |
|---|---:|
| Total Orders | 99,441 |
| Total Order Items | 112,650 |
| Merchandise Value | R$13,591,643.70 |
| Freight Value | R$2,251,909.54 |
| Unique Customers | 96,096 |
| Average Order Value | R$136.68 |
| On-Time Delivery Rate | 93.23% |
| Average Review Score | Approximately 4.09 |
| Total Payment Records | 103,886 |
| Distinct Orders Represented in Payments | 99,440 |
| Total Payment Value | R$16,008,872.12 |

The following additional checks were performed:

- Reporting-model grain and uniqueness validation.
- Merchandise value and freight reconciliation across compatible models.
- Payment-record count and payment-value reconciliation.
- Selected-category interaction testing.
- Reporting-period filter testing using narrower date ranges.
- Customer frequency and repeat-customer calculation checks.
- Delivery-review population and average-score reconciliation.

Small floating-point display differences may occur because some measures are stored or calculated using floating-point numeric types. Display precision is formatted for reporting; validation uses the underlying numeric results.

## 8. Visual Design and User Experience

The four dashboards follow a shared visual language while retaining layouts appropriate to their business purpose.

### Design standards

| Element | Design direction |
|---|---|
| Dashboard background | Light neutral grey |
| Worksheet background | White |
| Main title | Dark navy |
| Standard chart marks | Blue |
| Late delivery | Orange |
| On-Time delivery | Coral/red |
| Supporting text | Muted slate grey |
| Borders | Minimal or subtle |
| Dashboard dimensions | 1366 × 768 |
| Typography | Clean sans-serif |

### Design principles

- Prioritise business readability over decorative styling.
- Maintain consistent KPI presentation and title hierarchy.
- Use meaningful colour distinctions.
- Avoid unnecessary chart duplication.
- Preserve valid extreme observations.
- Keep chart axes and labels understandable.
- Provide detailed tooltips where full identifiers or supporting values are needed.
- Retain the correct analytical grain when improving visual presentation.

## 9. Analytical Limitations

The dashboards are descriptive and exploratory reporting tools. They do not establish causal relationships.

Important limitations include:

- The dataset represents historical Brazilian e-commerce activity, not current marketplace performance.
- Merchandise value does not provide profit or net-margin information.
- The dataset does not contain sufficient cost information for profitability analysis.
- Missing reviews cannot be interpreted as negative reviews.
- Delivery and review-score differences demonstrate association, not causation.
- State-level distinct customer counts are non-additive.
- Partial coverage and incomplete reporting periods must not be treated as fully comparable periods.
- No causal impact of marketing, retention or operational interventions is claimed.

## 10. Workbook Connection and Reproduction

### Development environment

- Tableau Desktop
- DuckDB database
- DuckDB Tableau connector
- DuckDB JDBC driver

### Database

The Tableau workbook was developed using the local DuckDB database:

`olist_analytics.duckdb`

The database contains the project's `clean`, `analytics` and `reporting` schemas.

### Reopening the workbook

A reviewer using the local workbook may need to:

1. Obtain the permitted project database or reproduce it using the documented SQL and source data.
2. Install a compatible Tableau Desktop version and the required DuckDB connector/driver.
3. Repoint the workbook to the local database location if the original machine-specific path is unavailable.
4. Confirm that the required schemas and views exist.
5. Refresh the Tableau data sources.
6. Test the reporting-period controls and D02 category interaction.

The local database path used during development is not portable by itself. Connection instructions must be followed when moving the workbook to another computer.

## 11. Screenshots and Publication

The final portfolio release should include screenshots of:

- D01 — Executive Overview
- D02 — Product & Category Performance
- D03 — Customer & Geographic Performance
- D04 — Marketplace Operations & Customer Experience

Recommended repository location:

```text
assets/
    tableau/
        D01_executive_overview.png
        D02_product_category_performance.png
        D03_customer_geographic_performance.png
        D04_marketplace_operations.png
```

Screenshots should show the complete dashboards at readable resolution and use a consistent reporting-period selection.

A Tableau Public publication link may be added here after the workbook is reviewed for data-sharing permission, privacy, portability and publication suitability.

**Publication status:** Update this section when the final workbook and screenshots have been released. Do not claim public publication before it has been completed.

## 12. Final Release Checklist

### Completed

- [x] Tableau reporting architecture documented.
- [x] Reporting-layer data sources and their grains documented.
- [x] D01 — Executive Overview built and functionally checked.
- [x] D02 — Product & Category Performance built and functionally checked.
- [x] D03 — Customer & Geographic Performance built and functionally checked.
- [x] D04 — Marketplace Operations & Customer Experience built and functionally checked.
- [x] Reporting-period controls tested across dashboards.
- [x] Dashboard calculations and key KPIs reconciled against validated SQL results.
- [x] Dashboard layouts and visual styling refined.
- [x] Dashboard interactions and relevant filters tested.

### Remaining Portfolio Release Tasks

- [ ] Complete the Business Insights & Recommendations report.
- [ ] Save and reopen the final Tableau workbook to verify the published version.
- [ ] Capture final dashboard screenshots.
- [ ] Add architecture and reporting-layer visuals where useful.
- [ ] Review and publish the relevant SQL and EDA notebook artifacts.
- [ ] Complete repository documentation and navigation.
- [ ] Prepare the final portfolio case study.

The Tableau dashboard development and functional validation phase is complete. Remaining tasks concern final documentation, presentation, and portfolio publication.
- [ ] Add a Tableau Public link if publication is appropriate.
- [ ] Complete the final portfolio release review.
