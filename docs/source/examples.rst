Examples
========

Five reference examples showing the range of difficulty and use cases
this dataset covers. Use these as a guide when writing your own submission.

.. contents::
   :local:
   :depth: 1

----

Category Revenue YoY Comparison
---------------------------------

**Difficulty:** Medium | **Tables:** fact_sales, dim_product, dim_date

**Business Question**

Which product categories generated the most revenue last quarter,
and how does that compare to the same quarter last year?

**Business Context**

Merchandising leadership preparing for annual planning meeting.
Need to identify growing vs declining categories YoY to guide
inventory investment, promotional spend, and category expansion decisions.

**KPIs**

.. list-table::
   :header-rows: 1

   * - KPI
     - Formula
   * - Category Revenue (Current Quarter)
     - ``SUM(net_sales) WHERE year=2024 AND quarter=4, grouped by category``
   * - Category Revenue (Prior Year Quarter)
     - ``SUM(net_sales) WHERE year=2023 AND quarter=4, grouped by category``
   * - YoY Revenue Change (Absolute)
     - ``revenue_q4_2024 - revenue_q4_2023``
   * - YoY Revenue Change (%)
     - ``ROUND(100.0 * delta / NULLIF(prior_year, 0), 2)``

**Key Technique**

Conditional aggregation with ``CASE WHEN`` pivots both years into one row per category.
``NULLIF`` guards division-by-zero for new categories with no prior year sales.

.. code-block:: sql

   SELECT
     p.category,
     SUM(CASE WHEN d.year = 2024 AND d.quarter = 4 THEN s.net_sales ELSE 0 END) AS revenue_q4_2024,
     SUM(CASE WHEN d.year = 2023 AND d.quarter = 4 THEN s.net_sales ELSE 0 END) AS revenue_q4_2023,
     SUM(CASE WHEN d.year = 2024 AND d.quarter = 4 THEN s.net_sales ELSE 0 END)
       - SUM(CASE WHEN d.year = 2023 AND d.quarter = 4 THEN s.net_sales ELSE 0 END) AS revenue_change,
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

**Difficulty:** Hard | **Tables:** fact_sales, dim_product, dim_date

**Business Question**

Based on the past three years of weekly sales, which weeks of the year show
the strongest and weakest sales patterns for each product category?

**Business Context**

Demand planning team building a seasonal sales calendar to guide procurement,
staffing, and promotional scheduling.

**KPIs**

.. list-table::
   :header-rows: 1

   * - KPI
     - Formula
   * - Average Weekly Revenue (3-Year)
     - ``AVG(SUM(net_sales))`` per category + iso_week across 3 prior years
   * - Annual Average Weekly Revenue per Category
     - ``AVG(avg_weekly_revenue_3yr)`` per category — seasonal baseline
   * - Seasonality Index
     - ``ROUND(avg_weekly / NULLIF(annual_avg, 0), 3)`` — >1.0 above average
   * - Peak / Trough Rank
     - ``RANK() OVER (PARTITION BY category ORDER BY avg_weekly DESC/ASC)``

**Key Technique**

4-CTE chain. CTE1 weekly revenue per category+year. CTE2 averages across 3 years.
CTE3 per-category annual baseline. CTE4 dual ``RANK()`` window functions.
Final ``WHERE`` keeps top-5 peak + bottom-5 trough weeks only.
Current year excluded — incomplete data skews averages.

.. code-block:: sql

   WITH weekly_category_sales AS (
     SELECT
       p.category,
       d.week AS iso_week,
       d.year,
       SUM(s.net_sales) AS weekly_revenue
     FROM fact_sales s
     JOIN dim_product p ON s.product_id = p.product_id
     JOIN dim_date d    ON s.date_id    = d.date_id
     WHERE s.is_return = FALSE
       AND d.year BETWEEN YEAR(CURRENT_DATE) - 3 AND YEAR(CURRENT_DATE) - 1
     GROUP BY p.category, d.week, d.year
   ),
   avg_by_week AS (
     SELECT category, iso_week,
       ROUND(AVG(weekly_revenue), 2) AS avg_weekly_revenue_3yr
     FROM weekly_category_sales
     GROUP BY category, iso_week
   ),
   category_annual_avg AS (
     SELECT category,
       ROUND(AVG(avg_weekly_revenue_3yr), 2) AS annual_avg_weekly_revenue
     FROM avg_by_week
     GROUP BY category
   ),
   indexed AS (
     SELECT
       abw.category, abw.iso_week, abw.avg_weekly_revenue_3yr,
       caa.annual_avg_weekly_revenue,
       ROUND(abw.avg_weekly_revenue_3yr / NULLIF(caa.annual_avg_weekly_revenue, 0), 3) AS seasonality_index,
       RANK() OVER (PARTITION BY abw.category ORDER BY abw.avg_weekly_revenue_3yr DESC) AS peak_rank,
       RANK() OVER (PARTITION BY abw.category ORDER BY abw.avg_weekly_revenue_3yr ASC)  AS trough_rank
     FROM avg_by_week abw
     JOIN category_annual_avg caa ON abw.category = caa.category
   )
   SELECT category, iso_week, avg_weekly_revenue_3yr,
     annual_avg_weekly_revenue, seasonality_index,
     CASE WHEN peak_rank <= 5 THEN 'Peak' WHEN trough_rank <= 5 THEN 'Trough' END AS week_classification
   FROM indexed
   WHERE peak_rank <= 5 OR trough_rank <= 5
   ORDER BY category, iso_week;

----

Slow Moving Inventory Stock Value
-----------------------------------

**Difficulty:** Medium | **Tables:** fact_sales, dim_product, dim_date

**Business Question**

Which products have been sitting in inventory for more than 60 days
without a sale, and what is their estimated stock value?

**Business Context**

Inventory management team runs monthly review to identify stagnant products.
Zero sales in 60+ days ties up capital, consumes warehouse space, risks obsolescence.
Report drives decisions on markdowns, clearance promotions, or supplier returns.

**KPIs**

.. list-table::
   :header-rows: 1

   * - KPI
     - Formula
   * - Days Since Last Sale
     - ``DATEDIFF(CURRENT_DATE, MAX(last_sale_date))`` per product_id
   * - Units Sold Last 90 Days (Stock Proxy)
     - ``SUM(quantity)`` last 90 days, non-returns, per product_id
   * - Estimated Stagnant Stock Value
     - ``ROUND(unit_cost * COALESCE(units_sold_last_90d, 0), 2)``

**Key Technique**

Two LEFT JOIN subqueries — one for all-time last sale date, one for 90-day volume.
No inventory quantity table exists — 90-day sales used as stock proxy.
``HAVING`` filters to products with no sale ever OR >60 days stagnant.
``COALESCE`` handles zero-sales products cleanly.

.. code-block:: sql

   SELECT
     p.product_id, p.product_name, p.category, p.sub_category,
     p.brand, p.unit_cost,
     COALESCE(recent.units_sold_last_90d, 0) AS units_sold_last_90d,
     MAX(last_sale.last_sale_date)            AS last_sale_date,
     DATEDIFF(CURRENT_DATE, MAX(last_sale.last_sale_date)) AS days_since_last_sale,
     ROUND(p.unit_cost * COALESCE(recent.units_sold_last_90d, 0), 2) AS estimated_stagnant_value
   FROM dim_product p
   LEFT JOIN (
     SELECT s.product_id, MAX(d.full_date) AS last_sale_date
     FROM fact_sales s JOIN dim_date d ON s.date_id = d.date_id
     WHERE s.is_return = FALSE
     GROUP BY s.product_id
   ) last_sale ON p.product_id = last_sale.product_id
   LEFT JOIN (
     SELECT s.product_id, SUM(s.quantity) AS units_sold_last_90d
     FROM fact_sales s JOIN dim_date d ON s.date_id = d.date_id
     WHERE s.is_return = FALSE
       AND d.full_date >= CURRENT_DATE - INTERVAL '90 days'
     GROUP BY s.product_id
   ) recent ON p.product_id = recent.product_id
   GROUP BY p.product_id, p.product_name, p.category, p.sub_category,
            p.brand, p.unit_cost, recent.units_sold_last_90d
   HAVING MAX(last_sale.last_sale_date) IS NULL
      OR DATEDIFF(CURRENT_DATE, MAX(last_sale.last_sale_date)) > 60
   ORDER BY estimated_stagnant_value DESC;

----

Store Return Rate Category Analysis
--------------------------------------

**Difficulty:** Hard | **Tables:** fact_sales, dim_store, dim_product, dim_date

**Business Question**

Which stores have a return rate above the company average,
and what product categories are driving those returns?

**Business Context**

Operations and QA team runs quarterly returns review. Stores with abnormally high
return rates flagged for investigation — causes may include product quality issues,
staff sales behaviour, or customer expectation misalignment.

**KPIs**

.. list-table::
   :header-rows: 1

   * - KPI
     - Formula
   * - Store Return Rate (%)
     - ``ROUND(100.0 * COUNT(is_return=TRUE) / COUNT(*), 2)`` per store, last 90 days
   * - Company Average Return Rate (%)
     - ``ROUND(AVG(return_rate_pct), 2)`` across all stores — benchmark
   * - Category Return Rate per High-Return Store
     - ``ROUND(100.0 * COUNT(is_return=TRUE) / COUNT(*), 2)`` per store + category
   * - Category Return Count
     - ``COUNT(is_return=TRUE)`` per store + category

**Key Technique**

3-CTE pattern. CTE1 per-store rates. CTE2 scalar company average via ``AVG()``.
CTE3 filters stores above average via ``CROSS JOIN``. Final query joins all dims,
groups by store+category. New stores excluded via ``opening_date`` filter to avoid
low-volume distortion.

.. code-block:: sql

   WITH store_return_rates AS (
     SELECT s.store_id,
       COUNT(CASE WHEN s.is_return = TRUE THEN 1 END) AS return_count,
       COUNT(*) AS total_transactions,
       ROUND(100.0 * COUNT(CASE WHEN s.is_return = TRUE THEN 1 END) / COUNT(*), 2) AS return_rate_pct
     FROM fact_sales s
     JOIN dim_date d ON s.date_id = d.date_id
     WHERE d.full_date >= CURRENT_DATE - INTERVAL '90 days'
     GROUP BY s.store_id
   ),
   company_avg AS (
     SELECT ROUND(AVG(return_rate_pct), 2) AS avg_return_rate FROM store_return_rates
   ),
   high_return_stores AS (
     SELECT srr.store_id FROM store_return_rates srr
     CROSS JOIN company_avg ca
     WHERE srr.return_rate_pct > ca.avg_return_rate
   )
   SELECT
     st.store_name, st.city, st.region, p.category,
     COUNT(CASE WHEN s.is_return = TRUE THEN 1 END) AS category_returns,
     COUNT(*) AS category_transactions,
     ROUND(100.0 * COUNT(CASE WHEN s.is_return = TRUE THEN 1 END) / COUNT(*), 2) AS category_return_rate_pct,
     ca.avg_return_rate AS company_avg_return_rate_pct
   FROM fact_sales s
   JOIN dim_store st   ON s.store_id   = st.store_id
   JOIN dim_product p  ON s.product_id = p.product_id
   JOIN dim_date d     ON s.date_id    = d.date_id
   CROSS JOIN company_avg ca
   WHERE s.store_id IN (SELECT store_id FROM high_return_stores)
     AND d.full_date >= CURRENT_DATE - INTERVAL '90 days'
     AND st.opening_date <= CURRENT_DATE - INTERVAL '90 days'
   GROUP BY st.store_name, st.city, st.region, p.category, ca.avg_return_rate
   ORDER BY st.store_name, category_return_rate_pct DESC;

----

Top Customer Spend by Acquisition Channel
-------------------------------------------

**Difficulty:** Hard | **Tables:** fact_sales, dim_customer, dim_date

**Business Question**

Who are our top 20% of customers by total spend in the past 12 months,
and what channels did they originally come from?

**Business Context**

CRM and loyalty team wants to identify high-value customers for a VIP retention program.
Acquisition channel breakdown helps marketing allocate budget toward channels producing
highest long-term customer value.

**KPIs**

.. list-table::
   :header-rows: 1

   * - KPI
     - Formula
   * - Total Spend per Customer (12 Months)
     - ``SUM(net_sales)`` non-returns, 12-month window, per customer_id
   * - Total Transactions per Customer
     - ``COUNT(DISTINCT sale_id)`` per customer_id, ``HAVING >= 2``
   * - Spend Quintile (Top 20% = Quintile 1)
     - ``NTILE(5) OVER (ORDER BY total_spend_12m DESC)``
   * - Average Order Value
     - ``ROUND(total_spend_12m / total_transactions, 2)``

**Key Technique**

2-CTE pattern. CTE1 aggregates spend + transactions (``HAVING >= 2`` excludes one-time buyers).
CTE2 applies ``NTILE(5)`` window function to bucket into spend quintiles.
Final query filters ``spend_quintile = 1``, joins ``dim_customer`` for channel + segment.
``avg_order_value`` included as secondary signal — high spend via few large orders vs many
small orders implies different retention strategies.

.. code-block:: sql

   WITH customer_spend AS (
     SELECT s.customer_id,
       SUM(s.net_sales)          AS total_spend_12m,
       COUNT(DISTINCT s.sale_id) AS total_transactions
     FROM fact_sales s
     JOIN dim_date d ON s.date_id = d.date_id
     WHERE s.is_return = FALSE
       AND d.full_date >= CURRENT_DATE - INTERVAL '12 months'
     GROUP BY s.customer_id
     HAVING COUNT(DISTINCT s.sale_id) >= 2
   ),
   ranked_customers AS (
     SELECT customer_id, total_spend_12m, total_transactions,
       NTILE(5) OVER (ORDER BY total_spend_12m DESC) AS spend_quintile
     FROM customer_spend
   )
   SELECT
     rc.customer_id,
     c.customer_segment,
     c.acquisition_channel,
     c.region,
     rc.total_spend_12m,
     rc.total_transactions,
     ROUND(rc.total_spend_12m / rc.total_transactions, 2) AS avg_order_value
   FROM ranked_customers rc
   JOIN dim_customer c ON rc.customer_id = c.customer_id
   WHERE rc.spend_quintile = 1
   ORDER BY rc.total_spend_12m DESC;
