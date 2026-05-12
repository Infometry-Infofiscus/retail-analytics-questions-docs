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

**SQL**

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

**Raw JSON Submission**

.. code-block:: json

   {
     "q_id": 1,
     "difficulty": "Medium",
     "db_type": "Relational (SQL)",
     "domain": "Retail",
     "instruction": "Which product categories generated the most revenue last quarter, and how does that compare to the same quarter last year?",
     "context": "Merchandising leadership preparing for annual planning meeting. Need to identify growing vs declining categories YoY to guide inventory investment, promotional spend, and category expansion decisions.",
     "metrics_and_aggregation": [
       {"kpi_metric_name": "Category Revenue (Current Quarter)", "aggregation_formula": "SUM(net_sales) WHERE year=2024 AND quarter=4, grouped by category"},
       {"kpi_metric_name": "Category Revenue (Prior Year Quarter)", "aggregation_formula": "SUM(net_sales) WHERE year=2023 AND quarter=4, grouped by category"},
       {"kpi_metric_name": "YoY Revenue Change (Absolute)", "aggregation_formula": "revenue_q4_2024 - revenue_q4_2023"},
       {"kpi_metric_name": "YoY Revenue Change (%)", "aggregation_formula": "ROUND(100.0 * delta / NULLIF(prior_year, 0), 2)"}
     ],
     "chain_of_thought": [
       "Step 1: Rank product categories by net revenue for Q4 2024 and compare against Q4 2023.",
       "Step 2: Need fact_sales, dim_product, dim_date.",
       "Step 3: Filter Q4 only, years 2023 and 2024. Exclude returns (is_return = FALSE).",
       "Step 4: Use conditional aggregation CASE WHEN to pivot both years into same row per category.",
       "Step 5: Order by Q4 2024 revenue descending to surface top performers first.",
       "Step 6: NULLIF guards division-by-zero for new categories with no prior year sales.",
       "Step 7: Only completed transactions counted. Returns excluded at filter level."
     ],
     "schema_tables": {
       "fact_tables": ["fact_sales"],
       "dimension_tables": ["dim_product", "dim_date"]
     },
     "data_model_layers": {
       "hierarchies": "Date > Quarter > Year, Product > Category",
       "aggregations": "agg_quarterly_revenue_by_category",
       "snapshots": ""
     },
     "sql": ""
   }

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

**SQL**

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

**Raw JSON Submission**

.. code-block:: json

   {
     "q_id": 2,
     "difficulty": "Hard",
     "db_type": "Relational (SQL)",
     "domain": "Retail",
     "instruction": "Based on the past three years of weekly sales, which weeks of the year show the strongest and weakest sales patterns for each product category?",
     "context": "Demand planning team building a seasonal sales calendar to guide procurement, staffing, and promotional scheduling. Identifying historically strong and weak weeks per category enables proactive stock alignment and marketing campaign timing.",
     "metrics_and_aggregation": [
       {"kpi_metric_name": "Average Weekly Revenue (3-Year)", "aggregation_formula": "AVG(SUM(net_sales)) grouped by category, iso_week across 3 prior years"},
       {"kpi_metric_name": "Annual Average Weekly Revenue per Category", "aggregation_formula": "AVG(avg_weekly_revenue_3yr) grouped by category — seasonal baseline"},
       {"kpi_metric_name": "Seasonality Index", "aggregation_formula": "ROUND(avg_weekly_revenue_3yr / NULLIF(annual_avg_weekly_revenue, 0), 3)"},
       {"kpi_metric_name": "Peak / Trough Rank", "aggregation_formula": "RANK() OVER (PARTITION BY category ORDER BY avg_weekly_revenue_3yr DESC/ASC)"}
     ],
     "chain_of_thought": [
       "Step 1: Build seasonal demand index per product category by ISO week — top 5 peak and bottom 5 trough weeks.",
       "Step 2: Need fact_sales for net revenue, dim_product for category, dim_date for ISO week and year.",
       "Step 3: Filter non-returns, 3 prior complete years. Current year excluded — incomplete data skews averages.",
       "Step 4: CTE1 weekly revenue. CTE2 3-year average per category+week. CTE3 annual baseline. CTE4 dual RANK().",
       "Step 5: Final SELECT filters peak_rank <= 5 OR trough_rank <= 5.",
       "Step 6: NULLIF guards division-by-zero. Categories with fewer than 3 years still valid but less stable.",
       "Step 7: Seasonality index is relative signal — layer absolute revenue alongside for planning decisions."
     ],
     "schema_tables": {
       "fact_tables": ["fact_sales"],
       "dimension_tables": ["dim_product", "dim_date"]
     },
     "data_model_layers": {
       "hierarchies": "Date > ISO Week > Year, Product > Category",
       "aggregations": "agg_weekly_category_revenue_3yr, agg_seasonality_index",
       "snapshots": "snap_seasonal_calendar_annual"
     },
     "sql": ""
   }

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

**SQL**

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

**Raw JSON Submission**

.. code-block:: json

   {
     "q_id": 3,
     "difficulty": "Medium",
     "db_type": "Relational (SQL)",
     "domain": "Retail",
     "instruction": "Which products have been sitting in inventory for more than 60 days without a sale, and what is their estimated stock value?",
     "context": "Inventory management team runs monthly review to identify stagnant products. Zero sales in 60+ days ties up capital, consumes warehouse space, risks obsolescence. Report drives decisions on markdowns, clearance promotions, or supplier returns.",
     "metrics_and_aggregation": [
       {"kpi_metric_name": "Days Since Last Sale", "aggregation_formula": "DATEDIFF(CURRENT_DATE, MAX(last_sale_date)) per product_id"},
       {"kpi_metric_name": "Units Sold Last 90 Days (Stock Proxy)", "aggregation_formula": "SUM(quantity) WHERE full_date >= CURRENT_DATE - 90 days AND is_return = FALSE, grouped by product_id"},
       {"kpi_metric_name": "Estimated Stagnant Stock Value", "aggregation_formula": "ROUND(unit_cost * COALESCE(units_sold_last_90d, 0), 2) per product"}
     ],
     "chain_of_thought": [
       "Step 1: Surface all active products with no sales in 60+ days and estimate tied-up capital value.",
       "Step 2: Need dim_product, fact_sales, dim_date. No inventory table — use 90-day sales as stock proxy.",
       "Step 3: Filter non-returns only. Active products from dim_product. Two date windows needed.",
       "Step 4: Estimated stock value = unit_cost x units_sold_last_90d. COALESCE handles zero sales. ROUND to 2dp.",
       "Step 5: Order by estimated_stagnant_value DESC — highest capital-at-risk first.",
       "Step 6: HAVING filters last_sale_date IS NULL OR days_since_last_sale > 60. LEFT JOINs include zero-history products.",
       "Step 7: Stock value is a proxy not actual on-hand quantity — interpret with caution."
     ],
     "schema_tables": {
       "fact_tables": ["fact_sales"],
       "dimension_tables": ["dim_product", "dim_date"]
     },
     "data_model_layers": {
       "hierarchies": "Product > Sub-Category > Category > Brand",
       "aggregations": "agg_slow_moving_inventory_monthly",
       "snapshots": "snap_inventory_stagnant_60d"
     },
     "sql": ""
   }

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

**SQL**

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

**Raw JSON Submission**

.. code-block:: json

   {
     "q_id": 4,
     "difficulty": "Hard",
     "db_type": "Relational (SQL)",
     "domain": "Retail",
     "instruction": "Which stores have a return rate above the company average, and what product categories are driving those returns?",
     "context": "Operations and QA team runs quarterly returns review. Stores with abnormally high return rates flagged for investigation. Report drives coaching conversations with store managers and product team escalations.",
     "metrics_and_aggregation": [
       {"kpi_metric_name": "Store Return Rate (%)", "aggregation_formula": "ROUND(100.0 * COUNT(CASE WHEN is_return = TRUE THEN 1 END) / COUNT(*), 2) grouped by store_id, last 90 days"},
       {"kpi_metric_name": "Company Average Return Rate (%)", "aggregation_formula": "ROUND(AVG(return_rate_pct), 2) across all stores — benchmark"},
       {"kpi_metric_name": "Category Return Rate (%) per High-Return Store", "aggregation_formula": "ROUND(100.0 * COUNT(CASE WHEN is_return = TRUE THEN 1 END) / COUNT(*), 2) grouped by store_id, category"},
       {"kpi_metric_name": "Category Return Count", "aggregation_formula": "COUNT(CASE WHEN is_return = TRUE THEN 1 END) grouped by store_id, category"}
     ],
     "chain_of_thought": [
       "Step 1: Identify stores exceeding company-average return rate and drill into which categories drive returns.",
       "Step 2: Need fact_sales, dim_store, dim_product, dim_date.",
       "Step 3: Filter last 90 days. Exclude newly opened stores via opening_date filter.",
       "Step 4: CTE1 per-store rates. CTE2 company average scalar. CTE3 filters stores above average via CROSS JOIN.",
       "Step 5: Final SELECT joins high_return_stores back to all dims, groups by store+category.",
       "Step 6: CROSS JOIN passes benchmark into every output row. Stores with few transactions may have volatile rates.",
       "Step 7: Return rate computed on all transactions including returns in denominator — industry standard."
     ],
     "schema_tables": {
       "fact_tables": ["fact_sales"],
       "dimension_tables": ["dim_store", "dim_product", "dim_date"]
     },
     "data_model_layers": {
       "hierarchies": "Store > City > Region, Product > Category",
       "aggregations": "agg_store_return_rate_quarterly, agg_category_return_rate",
       "snapshots": "snap_store_returns_90d"
     },
     "sql": ""
   }

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

**SQL**

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

**Raw JSON Submission**

.. code-block:: json

   {
     "q_id": 5,
     "difficulty": "Hard",
     "db_type": "Relational (SQL)",
     "domain": "Retail",
     "instruction": "Who are our top 20% of customers by total spend in the past 12 months, and what channels did they originally come from?",
     "context": "CRM and loyalty team wants to identify high-value customers for a VIP retention program. Acquisition channel breakdown helps marketing allocate budget toward channels producing highest long-term customer value.",
     "metrics_and_aggregation": [
       {"kpi_metric_name": "Total Spend per Customer (12 Months)", "aggregation_formula": "SUM(net_sales) WHERE is_return = FALSE AND full_date >= CURRENT_DATE - 12 months, grouped by customer_id"},
       {"kpi_metric_name": "Total Transactions per Customer", "aggregation_formula": "COUNT(DISTINCT sale_id) per customer_id, HAVING >= 2"},
       {"kpi_metric_name": "Spend Quintile (Top 20% = Quintile 1)", "aggregation_formula": "NTILE(5) OVER (ORDER BY total_spend_12m DESC)"},
       {"kpi_metric_name": "Average Order Value", "aggregation_formula": "ROUND(total_spend_12m / total_transactions, 2) per customer"}
     ],
     "chain_of_thought": [
       "Step 1: Isolate top 20% customers by 12-month net spend and trace back original acquisition channel.",
       "Step 2: Need fact_sales for spend aggregation, dim_date for 12-month window, dim_customer for channel attributes.",
       "Step 3: Filter non-returns, rolling 12-month window. Exclude one-time buyers via HAVING COUNT >= 2.",
       "Step 4: CTE1 aggregates total net sales and transaction count. CTE2 applies NTILE(5) — quintile 1 = top 20%.",
       "Step 5: Final SELECT filters spend_quintile = 1, joins dim_customer, orders by total_spend_12m DESC.",
       "Step 6: NTILE splits evenly — borderline customers may sit just inside/outside quintile 1.",
       "Step 7: avg_order_value as secondary signal — few large orders vs many small orders = different retention strategy."
     ],
     "schema_tables": {
       "fact_tables": ["fact_sales"],
       "dimension_tables": ["dim_customer", "dim_date"]
     },
     "data_model_layers": {
       "hierarchies": "Customer > Segment > Region, Date > Month > Year",
       "aggregations": "agg_customer_spend_12m, agg_customer_transactions",
       "snapshots": "snap_top_customers_quarterly"
     },
     "sql": ""
   }
