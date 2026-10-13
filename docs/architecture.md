# Data Architecture and Grain Strategy

## Why the project does not use one giant flat table

The Olist source contains multiple one-to-many relationships. One order can have multiple items, payment records, sellers and review records. Joining all of these relationships into a universal table can multiply rows and duplicate additive measures.

The project therefore separates models by analytical grain.

## Layered architecture

```text
Raw source tables
      ↓
clean.*
      ↓
analytics.*
      ↓
reporting.*
      ↓
Tableau
```

## Core grain decisions

### `analytics.order_analysis_base`
**Grain:** one row per order.

Safe uses:
- total orders
- order-level merchandise value
- customer analysis
- delivery performance
- order-level review analysis
- customer geography

### `analytics.item_analysis_base`
**Grain:** one row per order item.

Safe uses:
- product performance
- category item volume
- seller commercial performance

Order-level payments and reviews are not repeated directly at item grain.

### `reporting.order_category_base`
**Grain:** one row per `(order_id, analytical_category_name)`.

Item rows are aggregated before order-level review and delivery context is attached.

### `reporting.order_seller_base`
**Grain:** one row per `(order_id, seller_id)`.

This prevents seller review measures from being weighted by item count.

### `reporting.payment_analysis_base`
**Grain:** one row per payment record, identified by `(order_id, payment_sequential)`.

Payment value is additive at this grain. Order-level merchandise value is intentionally excluded to prevent duplication when an order has multiple payment records.

### `reporting.state_marketplace_summary`
**Grain:** one row per state.

Customer-side and seller-side measures are aggregated independently before they are combined.

## Validation philosophy

A transformation is not considered reliable only because its input tables were validated earlier. Each new grain-changing transformation is reconciled against known control totals.

Examples:
- order-category model reconstructs **112,650 items** and **R$13,591,643.70 merchandise value**
- order-seller model reconstructs **112,650 items** and the same merchandise-value total
- payment model reconstructs **103,886 payment rows** and **R$16,008,872.12 payment value**
- state summary reconstructs **99,441 customer orders** and **112,650 seller-side items**

The objective is analytical confidence with purposeful tests, not maximum test count.
