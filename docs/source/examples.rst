Examples
========

Five reference examples live in ``examples/``. Study these before contributing.

.. contents::
   :local:
   :depth: 1

Category Revenue YoY Comparison
---------------------------------

**File:** ``retail_category_revenue_yoy_comparison.json``

**Difficulty:** Medium | **Tables:** fact_sales, dim_product, dim_date

*Which product categories generated the most revenue last quarter, and how does that compare to the same quarter last year?*

**KPIs:**

- Category Revenue (Current Quarter) — ``SUM(net_sales) WHERE year=2024 AND quarter=4``
- Category Revenue (Prior Year Quarter) — ``SUM(net_sales) WHERE year=2023 AND quarter=4``
- YoY Revenue Change (Absolute) — ``revenue_q4_2024 - revenue_q4_2023``
- YoY Revenue Change (%) — ``ROUND(100.0 * delta / NULLIF(prior, 0), 2)``

**Key technique:** Conditional aggregation with ``CASE WHEN`` pivots both years into one row per category. ``NULLIF`` guards division-by-zero for new categories.

.. code-block:: sql

   SELECT
     p.category,
     SUM(CASE WHEN d.year = 2024 AND d.quarter = 4 THEN s.net_sales ELSE 0 END) AS revenue_q4_2024,
     SUM(CASE WHEN d.year = 2023 AND d.quarter = 4 THEN s.net_sales ELSE 0 END) AS revenue_q4_2023,
     ROUND(
       100.0 * (
         SUM(CASE WHEN d.year = 2024 AND d.quarter = 4 THEN s.net_sales ELSE 0 END)
         - SUM(CASE WHEN d.year = 2023 AND d.quarter = 4 THEN s.net_sales ELSE 0 END)
       ) / NULLIF(SUM(CASE WHEN d.year = 2023 AND d.quarter = 4 THEN s.net_sales ELSE 0 END), 0),
     2) AS yoy_change_pct
   FROM fact_sales s
   JOIN dim_product p ON s.product_id = p.product_id
   JOIN dim_date d    ON s.date_id    = d.date_id
   WHERE s.is_return = FALSE
     AND d.year IN (2023, 2024)
     AND d.quarter = 4
   GROUP BY p.category
   ORDER BY revenue_q4_2024 DESC;

----

Seasonal Weekly Sales Pattern by Category
-------------------------------------------

**File:** ``retail_seasonal_weekly_sales_pattern_by_category.json``

**Difficulty:** Hard | **Tables:** fact_sales, dim_product, dim_date

*Based on the past three years of weekly sales, which weeks show the strongest and weakest sales patterns for each product category?*

**KPIs:**

- Average Weekly Revenue (3-Year) — ``AVG(SUM(net_sales))`` per category + iso_week across 3 prior years
- Seasonality Index — ``avg_weekly_revenue_3yr / NULLIF(annual_avg_weekly_revenue, 0)``
- Peak / Trough Rank — ``RANK() OVER (PARTITION BY category ORDER BY avg_weekly_revenue_3yr DESC/ASC)``

**Key technique:** 4-CTE chain. CTE1 aggregates weekly. CTE2 averages across years. CTE3 computes per-category annual baseline. CTE4 applies dual ``RANK()`` window functions. Final ``WHERE`` filters to top-5 peak + bottom-5 trough weeks per category.

----

Slow Moving Inventory Stock Value
-----------------------------------

**File:** ``retail_slow_moving_inventory_stock_value.json``

**Difficulty:** Medium | **Tables:** fact_sales, dim_product, dim_date

*Which products have been sitting in inventory for more than 60 days without a sale, and what is their estimated stock value?*

**KPIs:**

- Days Since Last Sale — ``DATEDIFF(CURRENT_DATE, MAX(last_sale_date))``
- Units Sold Last 90 Days (Stock Proxy) — ``SUM(quantity)`` last 90 days, non-returns
- Estimated Stagnant Stock Value — ``ROUND(unit_cost * COALESCE(units_sold_last_90d, 0), 2)``

**Key technique:** Two LEFT JOIN subqueries — one for all-time last sale date, one for 90-day volume. ``HAVING`` filters to products with no sale OR >60 days stagnant. ``COALESCE`` handles zero-sales products.

----

Store Return Rate Category Analysis
--------------------------------------

**File:** ``retail_store_return_rate_category_analysis.json``

**Difficulty:** Hard | **Tables:** fact_sales, dim_store, dim_product, dim_date

*Which stores have a return rate above the company average, and what product categories are driving those returns?*

**KPIs:**

- Store Return Rate (%) — ``ROUND(100.0 * COUNT(is_return=TRUE) / COUNT(*), 2)`` per store
- Company Average Return Rate — ``ROUND(AVG(return_rate_pct), 2)`` across all stores
- Category Return Rate per High-Return Store

**Key technique:** 3-CTE pattern. CTE1 per-store rates. CTE2 scalar company average. CTE3 filters stores above average. ``CROSS JOIN`` passes benchmark into every output row. New stores excluded via ``opening_date`` filter.

----

Top Customer Spend by Acquisition Channel
-------------------------------------------

**File:** ``retail_top_customer_spend_acquisition_channel.json``

**Difficulty:** Hard | **Tables:** fact_sales, dim_customer, dim_date

*Who are our top 20% of customers by total spend in the past 12 months, and what channels did they originally come from?*

**KPIs:**

- Total Spend per Customer (12 Months) — ``SUM(net_sales)`` non-returns, 12-month window
- Spend Quintile — ``NTILE(5) OVER (ORDER BY total_spend_12m DESC)``
- Average Order Value — ``ROUND(total_spend / total_transactions, 2)``

**Key technique:** 2-CTE pattern. CTE1 aggregates spend + transactions (``HAVING >= 2`` excludes one-time buyers). CTE2 applies ``NTILE(5)`` window function. Final query filters ``spend_quintile = 1`` and joins ``dim_customer`` for channel/segment attributes.
