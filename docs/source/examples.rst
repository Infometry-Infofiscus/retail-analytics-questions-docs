Examples
========

Reference examples showing the range of difficulty, domains, and use cases
this dataset covers. Each domain includes real-world business scenarios with
full KPI definitions, chain of thought, and SQL. Use these as a guide when
writing your own submission.

.. contents::
   :local:
   :depth: 1

----

Retail
======

.. _example-category-revenue-yoy:

Category Revenue YoY Comparison
---------------------------------

**Difficulty:** Medium | **Domain:** Retail | **Tables:** fact_sales, dim_product, dim_date

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
     "sql": "SELECT p.category, SUM(CASE WHEN d.year = 2024 AND d.quarter = 4 THEN s.net_sales ELSE 0 END) AS revenue_q4_2024, SUM(CASE WHEN d.year = 2023 AND d.quarter = 4 THEN s.net_sales ELSE 0 END) AS revenue_q4_2023, SUM(CASE WHEN d.year = 2024 AND d.quarter = 4 THEN s.net_sales ELSE 0 END) - SUM(CASE WHEN d.year = 2023 AND d.quarter = 4 THEN s.net_sales ELSE 0 END) AS revenue_change, ROUND(100.0 * (SUM(CASE WHEN d.year = 2024 AND d.quarter = 4 THEN s.net_sales ELSE 0 END) - SUM(CASE WHEN d.year = 2023 AND d.quarter = 4 THEN s.net_sales ELSE 0 END)) / NULLIF(SUM(CASE WHEN d.year = 2023 AND d.quarter = 4 THEN s.net_sales ELSE 0 END), 0), 2) AS yoy_change_pct FROM fact_sales s JOIN dim_product p ON s.product_id = p.product_id JOIN dim_date d ON s.date_id = d.date_id WHERE s.is_return = FALSE AND d.year IN (2023, 2024) AND d.quarter = 4 GROUP BY p.category ORDER BY revenue_q4_2024 DESC;"
   }

----

.. _example-slow-moving-inventory:

Slow Moving Inventory Stock Value
-----------------------------------

**Difficulty:** Medium | **Domain:** Retail | **Tables:** fact_sales, dim_product, dim_date

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

**Raw JSON Submission**

.. code-block:: json

   {
     "q_id": 3,
     "difficulty": "Medium",
     "db_type": "Relational (SQL)",
     "domain": "Retail",
     "instruction": "Which products have been sitting in inventory for more than 60 days without a sale, and what is their estimated stock value?",
     "context": "Inventory management team runs monthly review to identify stagnant products. Zero sales in 60+ days ties up capital, consumes warehouse space, risks obsolescence.",
     "metrics_and_aggregation": [
       {"kpi_metric_name": "Days Since Last Sale", "aggregation_formula": "DATEDIFF(CURRENT_DATE, MAX(last_sale_date)) per product_id"},
       {"kpi_metric_name": "Units Sold Last 90 Days (Stock Proxy)", "aggregation_formula": "SUM(quantity) WHERE full_date >= CURRENT_DATE - 90 days AND is_return = FALSE, grouped by product_id"},
       {"kpi_metric_name": "Estimated Stagnant Stock Value", "aggregation_formula": "ROUND(unit_cost * COALESCE(units_sold_last_90d, 0), 2) per product"}
     ],
     "chain_of_thought": [
       "Step 1: Surface all active products with no sales in 60+ days and estimate tied-up capital value.",
       "Step 2: Need dim_product, fact_sales, dim_date. No inventory table — use 90-day sales as stock proxy.",
       "Step 3: Filter non-returns only. Active products from dim_product. Two date windows needed.",
       "Step 4: LEFT JOINs include zero-history products. HAVING filters last_sale_date IS NULL OR days > 60.",
       "Step 5: Order by estimated_stagnant_value DESC — highest capital-at-risk first."
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
     "sql": "SELECT p.product_id, p.product_name, p.category, p.unit_cost, COALESCE(recent.units_sold_last_90d, 0) AS units_sold_last_90d, MAX(last_sale.last_sale_date) AS last_sale_date, DATEDIFF(CURRENT_DATE, MAX(last_sale.last_sale_date)) AS days_since_last_sale, ROUND(p.unit_cost * COALESCE(recent.units_sold_last_90d, 0), 2) AS estimated_stagnant_value FROM dim_product p LEFT JOIN (SELECT s.product_id, MAX(d.full_date) AS last_sale_date FROM fact_sales s JOIN dim_date d ON s.date_id = d.date_id WHERE s.is_return = FALSE GROUP BY s.product_id) last_sale ON p.product_id = last_sale.product_id LEFT JOIN (SELECT s.product_id, SUM(s.quantity) AS units_sold_last_90d FROM fact_sales s JOIN dim_date d ON s.date_id = d.date_id WHERE s.is_return = FALSE AND d.full_date >= CURRENT_DATE - INTERVAL '90 days' GROUP BY s.product_id) recent ON p.product_id = recent.product_id GROUP BY p.product_id, p.product_name, p.category, p.unit_cost, recent.units_sold_last_90d HAVING MAX(last_sale.last_sale_date) IS NULL OR DATEDIFF(CURRENT_DATE, MAX(last_sale.last_sale_date)) > 60 ORDER BY estimated_stagnant_value DESC;"
   }

----

Healthcare
==========

.. _example-hospital-readmission-rate:

Hospital 30-Day Readmission Rate by Department
------------------------------------------------

**Difficulty:** Medium | **Domain:** Healthcare | **Tables:** fact_encounters, dim_patient, dim_department, dim_date

**Business Question**

Which hospital departments have the highest 30-day patient readmission rates,
and how does this compare to the national benchmark of 15%?

**Business Context**

Quality assurance team running quarterly clinical outcomes review.
High readmission rates indicate gaps in discharge planning, post-care follow-up,
or treatment effectiveness. Departments above benchmark are flagged for intervention.

**KPIs**

.. list-table::
   :header-rows: 1

   * - KPI
     - Formula
   * - Total Discharges per Department
     - ``COUNT(DISTINCT encounter_id) WHERE encounter_type = 'discharge'``
   * - 30-Day Readmissions
     - ``COUNT(DISTINCT patient_id)`` readmitted within 30 days of discharge
   * - Readmission Rate (%)
     - ``ROUND(100.0 * readmissions / total_discharges, 2)`` per department
   * - Variance from Benchmark
     - ``readmission_rate_pct - 15.0``

**Key Technique**

Self-join on ``fact_encounters`` to detect readmissions within 30-day window.
``DATEDIFF`` between discharge date and next admission date per patient.
``HAVING`` filters departments above 15% benchmark.
National benchmark (15.0) passed as literal constant for variance calc.

**Raw JSON Submission**

.. code-block:: json

   {
     "q_id": 6,
     "difficulty": "Medium",
     "db_type": "PostgreSQL",
     "domain": "Healthcare",
     "instruction": "Which hospital departments have the highest 30-day patient readmission rates, and how does this compare to the national benchmark of 15%?",
     "context": "Quality assurance team running quarterly clinical outcomes review. High readmission rates indicate gaps in discharge planning. Departments above benchmark are flagged for intervention.",
     "metrics_and_aggregation": [
       {"kpi_metric_name": "Total Discharges per Department", "aggregation_formula": "COUNT(DISTINCT encounter_id) WHERE encounter_type = 'discharge', grouped by department_id"},
       {"kpi_metric_name": "30-Day Readmissions", "aggregation_formula": "COUNT(DISTINCT patient_id) WHERE next_admission_date <= discharge_date + 30 days"},
       {"kpi_metric_name": "Readmission Rate (%)", "aggregation_formula": "ROUND(100.0 * readmissions / total_discharges, 2) per department"},
       {"kpi_metric_name": "Variance from Benchmark", "aggregation_formula": "readmission_rate_pct - 15.0"}
     ],
     "chain_of_thought": [
       "Step 1: Find departments with 30-day readmission rate above 15% national benchmark.",
       "Step 2: Need fact_encounters (discharge + readmit events), dim_department, dim_patient, dim_date.",
       "Step 3: Self-join fact_encounters on patient_id to find next admission after each discharge.",
       "Step 4: DATEDIFF between discharge_date and next admission_date — flag if <= 30 days.",
       "Step 5: Aggregate by department. Compute rate. Subtract 15.0 for benchmark variance.",
       "Step 6: Order by readmission_rate_pct DESC — worst performers first."
     ],
     "schema_tables": {
       "fact_tables": ["fact_encounters"],
       "dimension_tables": ["dim_patient", "dim_department", "dim_date"]
     },
     "data_model_layers": {
       "hierarchies": "Department > Division > Hospital, Date > Month > Quarter",
       "aggregations": "agg_readmission_rate_quarterly",
       "snapshots": "snap_department_outcomes_monthly"
     },
     "sql": "WITH discharges AS (SELECT e.encounter_id, e.patient_id, e.department_id, d.full_date AS discharge_date FROM fact_encounters e JOIN dim_date d ON e.date_id = d.date_id WHERE e.encounter_type = 'discharge'), readmits AS (SELECT e.patient_id, MIN(d.full_date) AS next_admission_date, e.department_id FROM fact_encounters e JOIN dim_date d ON e.date_id = d.date_id WHERE e.encounter_type = 'admission' GROUP BY e.patient_id, e.department_id), dept_rates AS (SELECT dis.department_id, COUNT(DISTINCT dis.encounter_id) AS total_discharges, COUNT(DISTINCT CASE WHEN DATEDIFF(ra.next_admission_date, dis.discharge_date) <= 30 THEN dis.patient_id END) AS readmissions FROM discharges dis LEFT JOIN readmits ra ON dis.patient_id = ra.patient_id AND ra.next_admission_date > dis.discharge_date GROUP BY dis.department_id) SELECT dp.department_name, dp.division, dr.total_discharges, dr.readmissions, ROUND(100.0 * dr.readmissions / NULLIF(dr.total_discharges, 0), 2) AS readmission_rate_pct, ROUND(100.0 * dr.readmissions / NULLIF(dr.total_discharges, 0), 2) - 15.0 AS variance_from_benchmark FROM dept_rates dr JOIN dim_department dp ON dr.department_id = dp.department_id ORDER BY readmission_rate_pct DESC;"
   }

----

.. _example-claims-denial-rate:

Insurance Claims Denial Rate by Diagnosis Code
------------------------------------------------

**Difficulty:** Hard | **Domain:** Healthcare | **Tables:** fact_claims, dim_patient, dim_provider, dim_date

**Business Question**

Which diagnosis codes have the highest insurance claim denial rates,
and which providers are submitting the most denied claims?

**Business Context**

Revenue cycle management team investigating claim denial patterns.
High denial rates on specific diagnosis codes suggest documentation gaps,
coding errors, or payer-specific policy mismatches. Provider-level breakdown
helps target training and audit efforts.

**KPIs**

.. list-table::
   :header-rows: 1

   * - KPI
     - Formula
   * - Total Claims per Diagnosis Code
     - ``COUNT(claim_id)`` grouped by diagnosis_code
   * - Denied Claims
     - ``COUNT(claim_id) WHERE claim_status = 'denied'``
   * - Denial Rate (%)
     - ``ROUND(100.0 * denied / total, 2)`` per diagnosis_code
   * - Avg Denied Claim Value
     - ``ROUND(AVG(claim_amount) WHERE status = 'denied', 2)``
   * - Top Denying Provider Count
     - ``COUNT(claim_id) WHERE status = 'denied'`` grouped by provider_id

**Key Technique**

Two-level aggregation — diagnosis code level first, then provider drill-down.
``RANK()`` window function to surface top 5 providers per diagnosis code.
``HAVING denial_rate > 20`` filters to actionable codes only.

**Raw JSON Submission**

.. code-block:: json

   {
     "q_id": 7,
     "difficulty": "Hard",
     "db_type": "Snowflake",
     "domain": "Healthcare",
     "instruction": "Which diagnosis codes have the highest insurance claim denial rates, and which providers are submitting the most denied claims?",
     "context": "Revenue cycle management team investigating denial patterns to target documentation improvement and provider training.",
     "metrics_and_aggregation": [
       {"kpi_metric_name": "Total Claims per Diagnosis Code", "aggregation_formula": "COUNT(claim_id) grouped by diagnosis_code"},
       {"kpi_metric_name": "Denied Claims", "aggregation_formula": "COUNT(claim_id) WHERE claim_status = 'denied'"},
       {"kpi_metric_name": "Denial Rate (%)", "aggregation_formula": "ROUND(100.0 * denied_claims / total_claims, 2)"},
       {"kpi_metric_name": "Avg Denied Claim Value", "aggregation_formula": "ROUND(AVG(claim_amount) WHERE claim_status = 'denied', 2)"},
       {"kpi_metric_name": "Top Denying Provider Count", "aggregation_formula": "COUNT(claim_id) WHERE claim_status = 'denied', grouped by provider_id, RANK() per diagnosis_code"}
     ],
     "chain_of_thought": [
       "Step 1: Find diagnosis codes with >20% denial rate and surface top providers driving denials.",
       "Step 2: Need fact_claims, dim_provider, dim_patient, dim_date.",
       "Step 3: CTE1 — denial rate per diagnosis_code. Filter HAVING denial_rate > 20%.",
       "Step 4: CTE2 — provider-level denied claim counts for those diagnosis codes.",
       "Step 5: RANK() OVER (PARTITION BY diagnosis_code ORDER BY denied_count DESC) — top 5 providers.",
       "Step 6: Join provider dim for name and specialty. Order by denial_rate DESC."
     ],
     "schema_tables": {
       "fact_tables": ["fact_claims"],
       "dimension_tables": ["dim_patient", "dim_provider", "dim_date"]
     },
     "data_model_layers": {
       "hierarchies": "Provider > Specialty > Division, Date > Month > Year",
       "aggregations": "agg_denial_rate_by_diagnosis, agg_provider_denial_count",
       "snapshots": "snap_claims_denial_monthly"
     },
     "sql": "WITH diagnosis_denial AS (SELECT diagnosis_code, COUNT(claim_id) AS total_claims, COUNT(CASE WHEN claim_status = 'denied' THEN 1 END) AS denied_claims, ROUND(100.0 * COUNT(CASE WHEN claim_status = 'denied' THEN 1 END) / NULLIF(COUNT(claim_id), 0), 2) AS denial_rate_pct, ROUND(AVG(CASE WHEN claim_status = 'denied' THEN claim_amount END), 2) AS avg_denied_value FROM fact_claims GROUP BY diagnosis_code HAVING ROUND(100.0 * COUNT(CASE WHEN claim_status = 'denied' THEN 1 END) / NULLIF(COUNT(claim_id), 0), 2) > 20), provider_denials AS (SELECT fc.diagnosis_code, fc.provider_id, COUNT(fc.claim_id) AS provider_denied_count, RANK() OVER (PARTITION BY fc.diagnosis_code ORDER BY COUNT(fc.claim_id) DESC) AS denial_rank FROM fact_claims fc WHERE fc.claim_status = 'denied' AND fc.diagnosis_code IN (SELECT diagnosis_code FROM diagnosis_denial) GROUP BY fc.diagnosis_code, fc.provider_id) SELECT dd.diagnosis_code, dd.total_claims, dd.denied_claims, dd.denial_rate_pct, dd.avg_denied_value, p.provider_name, p.specialty, pd.provider_denied_count FROM diagnosis_denial dd JOIN provider_denials pd ON dd.diagnosis_code = pd.diagnosis_code AND pd.denial_rank <= 5 JOIN dim_provider p ON pd.provider_id = p.provider_id ORDER BY dd.denial_rate_pct DESC, pd.denial_rank;"
   }

----

HighTech (SaaS)
===============

.. _example-saas-churn-rate:

Monthly Churn Rate by Subscription Plan
-----------------------------------------

**Difficulty:** Medium | **Domain:** HighTech (SaaS) | **Tables:** fact_subscriptions, dim_customer, dim_plan, dim_date

**Business Question**

What is the monthly churn rate for each subscription plan tier over the past 6 months,
and which plan has the worst retention?

**Business Context**

Product and growth team tracking retention health across Free, Pro, and Enterprise tiers.
High churn in a specific tier signals pricing, onboarding, or feature-fit issues.
Monthly tracking enables early intervention before churn compounds.

**KPIs**

.. list-table::
   :header-rows: 1

   * - KPI
     - Formula
   * - Churned Customers per Month
     - ``COUNT(customer_id) WHERE status = 'churned'`` grouped by plan_tier, month
   * - Active Customers Start of Month
     - ``COUNT(customer_id) WHERE status = 'active'`` at month start, per plan_tier
   * - Monthly Churn Rate (%)
     - ``ROUND(100.0 * churned / active_start, 2)`` per plan_tier, per month
   * - Avg Days to Churn
     - ``AVG(DATEDIFF(churn_date, subscription_start_date))`` per plan_tier

**Key Technique**

``LAG()`` window function to get prior-month active count as denominator.
Filter ``subscription_end_date`` within each calendar month to identify churns.
Group by ``plan_tier`` and ``month`` for tier-level breakdown.
``Avg Days to Churn`` reveals whether churn is early (onboarding) or late (value).

**Raw JSON Submission**

.. code-block:: json

   {
     "q_id": 8,
     "difficulty": "Medium",
     "db_type": "BigQuery",
     "domain": "HighTech (SaaS)",
     "instruction": "What is the monthly churn rate for each subscription plan tier over the past 6 months, and which plan has the worst retention?",
     "context": "Product and growth team tracking retention health across Free, Pro, and Enterprise tiers. High churn in a specific tier signals pricing or onboarding issues.",
     "metrics_and_aggregation": [
       {"kpi_metric_name": "Churned Customers per Month", "aggregation_formula": "COUNT(customer_id) WHERE subscription_status = 'churned' AND churn_date within month, grouped by plan_tier, month"},
       {"kpi_metric_name": "Active Customers Start of Month", "aggregation_formula": "COUNT(customer_id) WHERE status = 'active' at first day of month, per plan_tier"},
       {"kpi_metric_name": "Monthly Churn Rate (%)", "aggregation_formula": "ROUND(100.0 * churned_count / NULLIF(active_start_count, 0), 2)"},
       {"kpi_metric_name": "Avg Days to Churn", "aggregation_formula": "AVG(DATEDIFF(churn_date, subscription_start_date)) per plan_tier"}
     ],
     "chain_of_thought": [
       "Step 1: Compute monthly churn rate per plan tier for last 6 months.",
       "Step 2: Need fact_subscriptions, dim_customer, dim_plan, dim_date.",
       "Step 3: CTE1 — active customers at start of each month per plan. CTE2 — churned customers per month per plan.",
       "Step 4: Join CTE1 and CTE2 on plan_tier + month. Compute churn rate.",
       "Step 5: Include avg_days_to_churn to distinguish early vs late churn patterns.",
       "Step 6: Order by month ASC, churn_rate_pct DESC per month."
     ],
     "schema_tables": {
       "fact_tables": ["fact_subscriptions"],
       "dimension_tables": ["dim_customer", "dim_plan", "dim_date"]
     },
     "data_model_layers": {
       "hierarchies": "Plan > Tier > Product, Date > Month > Quarter",
       "aggregations": "agg_monthly_churn_by_plan",
       "snapshots": "snap_subscription_health_monthly"
     },
     "sql": "WITH monthly_active AS (SELECT dp.plan_tier, FORMAT_DATE('%Y-%m', DATE_TRUNC(d.full_date, MONTH)) AS month, COUNT(DISTINCT fs.customer_id) AS active_start_count FROM fact_subscriptions fs JOIN dim_plan dp ON fs.plan_id = dp.plan_id JOIN dim_date d ON fs.date_id = d.date_id WHERE fs.subscription_status = 'active' AND d.full_date = DATE_TRUNC(d.full_date, MONTH) AND d.full_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 6 MONTH) GROUP BY dp.plan_tier, month), monthly_churn AS (SELECT dp.plan_tier, FORMAT_DATE('%Y-%m', fs.churn_date) AS month, COUNT(DISTINCT fs.customer_id) AS churned_count, ROUND(AVG(DATE_DIFF(fs.churn_date, fs.subscription_start_date, DAY)), 1) AS avg_days_to_churn FROM fact_subscriptions fs JOIN dim_plan dp ON fs.plan_id = dp.plan_id WHERE fs.subscription_status = 'churned' AND fs.churn_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 6 MONTH) GROUP BY dp.plan_tier, month) SELECT ma.plan_tier, ma.month, ma.active_start_count, COALESCE(mc.churned_count, 0) AS churned_count, ROUND(100.0 * COALESCE(mc.churned_count, 0) / NULLIF(ma.active_start_count, 0), 2) AS churn_rate_pct, COALESCE(mc.avg_days_to_churn, 0) AS avg_days_to_churn FROM monthly_active ma LEFT JOIN monthly_churn mc ON ma.plan_tier = mc.plan_tier AND ma.month = mc.month ORDER BY ma.month, churn_rate_pct DESC;"
   }

----

.. _example-saas-feature-adoption:

Feature Adoption Funnel by Customer Segment
---------------------------------------------

**Difficulty:** Hard | **Domain:** HighTech (SaaS) | **Tables:** fact_feature_usage, dim_customer, dim_feature, dim_date

**Business Question**

What percentage of customers in each segment have adopted our core features,
and where in the adoption funnel are they dropping off?

**Business Context**

Product team running quarterly feature adoption review. Low adoption of core features
correlates strongly with churn. Segment-level funnel reveals whether onboarding,
UX, or feature-market fit is the root cause.

**KPIs**

.. list-table::
   :header-rows: 1

   * - KPI
     - Formula
   * - Customers Who Tried Feature
     - ``COUNT(DISTINCT customer_id)`` per feature, per segment
   * - Adoption Rate (%)
     - ``ROUND(100.0 * tried / total_segment_customers, 2)``
   * - Repeat Usage Rate (%)
     - ``COUNT(DISTINCT customer_id WHERE usage_count >= 3) / tried``
   * - Funnel Drop-off Rate (%)
     - ``100 - repeat_usage_rate_pct``

**Key Technique**

3-CTE pattern. CTE1 total customers per segment baseline. CTE2 feature trial counts.
CTE3 repeat users (usage_count >= 3 threshold). Final join computes full funnel.
``RANK()`` surfaces features with worst drop-off per segment.

**Raw JSON Submission**

.. code-block:: json

   {
     "q_id": 9,
     "difficulty": "Hard",
     "db_type": "Snowflake",
     "domain": "HighTech (SaaS)",
     "instruction": "What percentage of customers in each segment have adopted our core features, and where in the adoption funnel are they dropping off?",
     "context": "Product team running quarterly feature adoption review. Low adoption of core features correlates with churn. Segment-level funnel reveals onboarding vs UX vs fit issues.",
     "metrics_and_aggregation": [
       {"kpi_metric_name": "Customers Who Tried Feature", "aggregation_formula": "COUNT(DISTINCT customer_id) per feature_id, per customer_segment"},
       {"kpi_metric_name": "Adoption Rate (%)", "aggregation_formula": "ROUND(100.0 * tried_count / total_segment_customers, 2)"},
       {"kpi_metric_name": "Repeat Usage Rate (%)", "aggregation_formula": "ROUND(100.0 * COUNT(DISTINCT customer_id WHERE usage_count >= 3) / tried_count, 2)"},
       {"kpi_metric_name": "Funnel Drop-off Rate (%)", "aggregation_formula": "100.0 - repeat_usage_rate_pct"}
     ],
     "chain_of_thought": [
       "Step 1: Build feature adoption funnel per customer segment — trial rate and repeat usage rate.",
       "Step 2: Need fact_feature_usage, dim_customer, dim_feature, dim_date.",
       "Step 3: CTE1 — total active customers per segment (baseline denominator).",
       "Step 4: CTE2 — distinct customers who used each feature at least once.",
       "Step 5: CTE3 — customers with usage_count >= 3 (repeat/sticky usage threshold).",
       "Step 6: Join all CTEs, compute adoption_rate and repeat_usage_rate.",
       "Step 7: RANK() per segment by drop-off_rate DESC — worst funnel leaks first."
     ],
     "schema_tables": {
       "fact_tables": ["fact_feature_usage"],
       "dimension_tables": ["dim_customer", "dim_feature", "dim_date"]
     },
     "data_model_layers": {
       "hierarchies": "Feature > Feature_Group > Product, Customer > Segment > Region",
       "aggregations": "agg_feature_adoption_by_segment, agg_repeat_usage_rate",
       "snapshots": "snap_feature_funnel_quarterly"
     },
     "sql": "WITH segment_baseline AS (SELECT c.customer_segment, COUNT(DISTINCT c.customer_id) AS total_customers FROM dim_customer c WHERE c.customer_status = 'active' GROUP BY c.customer_segment), feature_trials AS (SELECT fu.feature_id, c.customer_segment, COUNT(DISTINCT fu.customer_id) AS tried_count FROM fact_feature_usage fu JOIN dim_customer c ON fu.customer_id = c.customer_id GROUP BY fu.feature_id, c.customer_segment), repeat_users AS (SELECT fu.feature_id, c.customer_segment, COUNT(DISTINCT fu.customer_id) AS repeat_count FROM fact_feature_usage fu JOIN dim_customer c ON fu.customer_id = c.customer_id GROUP BY fu.feature_id, c.customer_segment HAVING COUNT(fu.usage_event_id) >= 3) SELECT df.feature_name, ft.customer_segment, sb.total_customers, ft.tried_count, ROUND(100.0 * ft.tried_count / NULLIF(sb.total_customers, 0), 2) AS adoption_rate_pct, COALESCE(ru.repeat_count, 0) AS repeat_count, ROUND(100.0 * COALESCE(ru.repeat_count, 0) / NULLIF(ft.tried_count, 0), 2) AS repeat_usage_rate_pct, ROUND(100.0 - ROUND(100.0 * COALESCE(ru.repeat_count, 0) / NULLIF(ft.tried_count, 0), 2), 2) AS dropoff_rate_pct, RANK() OVER (PARTITION BY ft.customer_segment ORDER BY ROUND(100.0 - ROUND(100.0 * COALESCE(ru.repeat_count, 0) / NULLIF(ft.tried_count, 0), 2), 2) DESC) AS dropoff_rank FROM feature_trials ft JOIN segment_baseline sb ON ft.customer_segment = sb.customer_segment JOIN dim_feature df ON ft.feature_id = df.feature_id LEFT JOIN repeat_users ru ON ft.feature_id = ru.feature_id AND ft.customer_segment = ru.customer_segment ORDER BY ft.customer_segment, dropoff_rate_pct DESC;"
   }

----

Finance
=======

.. _example-finance-portfolio-performance:

Portfolio Performance vs Benchmark
------------------------------------

**Difficulty:** Hard | **Domain:** Finance | **Tables:** fact_transactions, dim_asset, dim_portfolio, dim_date

**Business Question**

How has each investment portfolio performed against its benchmark index
over the past 12 months, and which asset classes are driving over- or under-performance?

**Business Context**

Investment operations team running quarterly portfolio review.
Portfolios consistently underperforming their benchmark trigger rebalancing decisions.
Asset-class breakdown identifies whether under-performance is concentrated or broad.

**KPIs**

.. list-table::
   :header-rows: 1

   * - KPI
     - Formula
   * - Portfolio Return (%)
     - ``ROUND(100.0 * (end_value - start_value) / NULLIF(start_value, 0), 2)``
   * - Benchmark Return (%)
     - From ``dim_portfolio.benchmark_return_12m``
   * - Alpha (Excess Return)
     - ``portfolio_return_pct - benchmark_return_pct``
   * - Asset Class Contribution
     - ``SUM(asset_return * weight)`` per asset_class within portfolio

**Key Technique**

Start/end value computed via ``FIRST_VALUE`` and ``LAST_VALUE`` window functions
partitioned by portfolio_id ordered by date.
Alpha = portfolio return minus benchmark constant from dim table.
Asset class contribution uses weighted return aggregation.

**Raw JSON Submission**

.. code-block:: json

   {
     "q_id": 10,
     "difficulty": "Hard",
     "db_type": "Redshift",
     "domain": "Finance",
     "instruction": "How has each investment portfolio performed against its benchmark index over the past 12 months, and which asset classes are driving over- or under-performance?",
     "context": "Investment operations team running quarterly portfolio review. Portfolios underperforming benchmark trigger rebalancing decisions.",
     "metrics_and_aggregation": [
       {"kpi_metric_name": "Portfolio Return (%)", "aggregation_formula": "ROUND(100.0 * (end_value - start_value) / NULLIF(start_value, 0), 2) per portfolio_id"},
       {"kpi_metric_name": "Benchmark Return (%)", "aggregation_formula": "dim_portfolio.benchmark_return_12m constant per portfolio"},
       {"kpi_metric_name": "Alpha (Excess Return)", "aggregation_formula": "portfolio_return_pct - benchmark_return_pct"},
       {"kpi_metric_name": "Asset Class Contribution", "aggregation_formula": "SUM(asset_return_pct * asset_weight) grouped by asset_class within portfolio"}
     ],
     "chain_of_thought": [
       "Step 1: Compute 12-month return per portfolio vs benchmark, drill into asset class contribution.",
       "Step 2: Need fact_transactions, dim_asset, dim_portfolio, dim_date.",
       "Step 3: CTE1 — start and end portfolio value using FIRST_VALUE/LAST_VALUE over 12-month window.",
       "Step 4: CTE2 — portfolio return % and alpha vs benchmark from dim_portfolio.",
       "Step 5: CTE3 — asset class weighted return contribution within each portfolio.",
       "Step 6: Order portfolios by alpha ASC — worst underperformers first for action."
     ],
     "schema_tables": {
       "fact_tables": ["fact_transactions"],
       "dimension_tables": ["dim_asset", "dim_portfolio", "dim_date"]
     },
     "data_model_layers": {
       "hierarchies": "Asset > Asset_Class > Portfolio, Date > Month > Quarter > Year",
       "aggregations": "agg_portfolio_return_12m, agg_asset_class_contribution",
       "snapshots": "snap_portfolio_performance_quarterly"
     },
     "sql": "WITH portfolio_values AS (SELECT t.portfolio_id, FIRST_VALUE(t.portfolio_value) OVER (PARTITION BY t.portfolio_id ORDER BY d.full_date ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS start_value, LAST_VALUE(t.portfolio_value) OVER (PARTITION BY t.portfolio_id ORDER BY d.full_date ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS end_value FROM fact_transactions t JOIN dim_date d ON t.date_id = d.date_id WHERE d.full_date >= CURRENT_DATE - INTERVAL '12 months' GROUP BY t.portfolio_id, t.portfolio_value, d.full_date), portfolio_returns AS (SELECT pv.portfolio_id, MAX(pv.start_value) AS start_value, MAX(pv.end_value) AS end_value, ROUND(100.0 * (MAX(pv.end_value) - MAX(pv.start_value)) / NULLIF(MAX(pv.start_value), 0), 2) AS portfolio_return_pct FROM portfolio_values pv GROUP BY pv.portfolio_id), asset_contribution AS (SELECT t.portfolio_id, a.asset_class, ROUND(SUM(t.asset_return_pct * t.asset_weight), 4) AS class_contribution FROM fact_transactions t JOIN dim_asset a ON t.asset_id = a.asset_id JOIN dim_date d ON t.date_id = d.date_id WHERE d.full_date >= CURRENT_DATE - INTERVAL '12 months' GROUP BY t.portfolio_id, a.asset_class) SELECT p.portfolio_name, pr.portfolio_return_pct, p.benchmark_return_12m AS benchmark_return_pct, ROUND(pr.portfolio_return_pct - p.benchmark_return_12m, 2) AS alpha, ac.asset_class, ac.class_contribution FROM portfolio_returns pr JOIN dim_portfolio p ON pr.portfolio_id = p.portfolio_id JOIN asset_contribution ac ON pr.portfolio_id = ac.portfolio_id ORDER BY alpha ASC, ac.class_contribution ASC;"
   }

----

Manufacturing
=============

.. _example-manufacturing-oee:

Overall Equipment Effectiveness (OEE) by Production Line
----------------------------------------------------------

**Difficulty:** Hard | **Domain:** Manufacturing | **Tables:** fact_production, dim_equipment, dim_shift, dim_date

**Business Question**

What is the Overall Equipment Effectiveness (OEE) score for each production line
over the past quarter, and which component — availability, performance, or quality —
is dragging the score down?

**Business Context**

Plant operations manager running monthly equipment utilization review.
OEE below 85% (world-class threshold) triggers targeted maintenance or process audits.
Component breakdown (availability vs performance vs quality) directs the right fix.

**KPIs**

.. list-table::
   :header-rows: 1

   * - KPI
     - Formula
   * - Availability (%)
     - ``(planned_time - downtime) / planned_time * 100``
   * - Performance (%)
     - ``(actual_output / theoretical_max_output) * 100``
   * - Quality (%)
     - ``(good_units / total_units_produced) * 100``
   * - OEE Score (%)
     - ``ROUND(availability * performance * quality / 10000, 2)``
   * - Gap to World-Class (85%)
     - ``85.0 - oee_score``

**Key Technique**

All three OEE components computed in single CTE using conditional aggregation.
OEE = product of three ratios (multiplied, not averaged).
``CASE WHEN`` identifies the weakest component per line for root cause routing.
Lines below 85% flagged with ``gap_to_worldclass`` for prioritization.

**Raw JSON Submission**

.. code-block:: json

   {
     "q_id": 11,
     "difficulty": "Hard",
     "db_type": "Azure Synapse",
     "domain": "Manufacturing",
     "instruction": "What is the OEE score for each production line over the past quarter, and which component is dragging the score down?",
     "context": "Plant operations manager reviewing equipment utilization. OEE below 85% triggers maintenance or process audit. Component breakdown directs the right fix.",
     "metrics_and_aggregation": [
       {"kpi_metric_name": "Availability (%)", "aggregation_formula": "ROUND(100.0 * SUM(planned_time - downtime_minutes) / NULLIF(SUM(planned_time), 0), 2) per equipment_line"},
       {"kpi_metric_name": "Performance (%)", "aggregation_formula": "ROUND(100.0 * SUM(actual_output) / NULLIF(SUM(theoretical_max_output), 0), 2)"},
       {"kpi_metric_name": "Quality (%)", "aggregation_formula": "ROUND(100.0 * SUM(good_units) / NULLIF(SUM(total_units), 0), 2)"},
       {"kpi_metric_name": "OEE Score (%)", "aggregation_formula": "ROUND(availability_pct * performance_pct * quality_pct / 10000.0, 2)"},
       {"kpi_metric_name": "Gap to World-Class", "aggregation_formula": "85.0 - oee_score"}
     ],
     "chain_of_thought": [
       "Step 1: Compute OEE per production line for last quarter. Flag lines below 85%.",
       "Step 2: Need fact_production, dim_equipment, dim_shift, dim_date.",
       "Step 3: CTE1 — compute availability, performance, quality per equipment_line in one pass.",
       "Step 4: CTE2 — multiply three ratios for OEE score. Compute gap to 85% world-class.",
       "Step 5: CASE WHEN identifies weakest component (min of three) for root cause routing.",
       "Step 6: Order by oee_score ASC — worst performing lines first."
     ],
     "schema_tables": {
       "fact_tables": ["fact_production"],
       "dimension_tables": ["dim_equipment", "dim_shift", "dim_date"]
     },
     "data_model_layers": {
       "hierarchies": "Equipment > Production_Line > Plant, Shift > Date > Quarter",
       "aggregations": "agg_oee_quarterly_by_line, agg_oee_components",
       "snapshots": "snap_equipment_oee_monthly"
     },
     "sql": "WITH oee_components AS (SELECT e.equipment_line, e.plant_name, ROUND(100.0 * SUM(p.planned_time_minutes - p.downtime_minutes) / NULLIF(SUM(p.planned_time_minutes), 0), 2) AS availability_pct, ROUND(100.0 * SUM(p.actual_output) / NULLIF(SUM(p.theoretical_max_output), 0), 2) AS performance_pct, ROUND(100.0 * SUM(p.good_units) / NULLIF(SUM(p.total_units_produced), 0), 2) AS quality_pct FROM fact_production p JOIN dim_equipment e ON p.equipment_id = e.equipment_id JOIN dim_date d ON p.date_id = d.date_id WHERE d.quarter = DATEPART(QUARTER, DATEADD(QUARTER, -1, CURRENT_DATE)) AND d.year = YEAR(DATEADD(QUARTER, -1, CURRENT_DATE)) GROUP BY e.equipment_line, e.plant_name), oee_scores AS (SELECT equipment_line, plant_name, availability_pct, performance_pct, quality_pct, ROUND(availability_pct * performance_pct * quality_pct / 10000.0, 2) AS oee_score, ROUND(85.0 - ROUND(availability_pct * performance_pct * quality_pct / 10000.0, 2), 2) AS gap_to_worldclass, CASE WHEN LEAST(availability_pct, performance_pct, quality_pct) = availability_pct THEN 'Availability' WHEN LEAST(availability_pct, performance_pct, quality_pct) = performance_pct THEN 'Performance' ELSE 'Quality' END AS weakest_component FROM oee_components) SELECT plant_name, equipment_line, availability_pct, performance_pct, quality_pct, oee_score, gap_to_worldclass, weakest_component FROM oee_scores ORDER BY oee_score ASC;"
   }

----

Supply Chain
============

.. _example-supplychain-supplier-lead-time:

Supplier Lead Time Variance Analysis
--------------------------------------

**Difficulty:** Medium | **Domain:** Supply Chain | **Tables:** fact_purchase_orders, dim_supplier, dim_product, dim_date

**Business Question**

Which suppliers consistently deliver outside their agreed lead time windows,
and what is the financial impact of late deliveries on production schedules?

**Business Context**

Procurement team running quarterly supplier performance review.
Suppliers with high lead time variance cause stock-outs, production delays,
and emergency procurement costs. SLA breach rate drives contract renegotiation decisions.

**KPIs**

.. list-table::
   :header-rows: 1

   * - KPI
     - Formula
   * - Agreed Lead Time (days)
     - ``dim_supplier.agreed_lead_time_days``
   * - Actual Lead Time (days)
     - ``DATEDIFF(actual_delivery_date, order_date)`` per PO
   * - Lead Time Variance (days)
     - ``actual_lead_time - agreed_lead_time``
   * - SLA Breach Rate (%)
     - ``ROUND(100.0 * late_orders / total_orders, 2)`` per supplier
   * - Avg Late Delivery Cost
     - ``AVG(emergency_procurement_cost) WHERE is_late = TRUE``

**Key Technique**

``DATEDIFF`` per PO for actual lead time. Compare to ``agreed_lead_time_days`` from dim.
``CASE WHEN actual > agreed THEN 1 END`` flags late orders.
Aggregate breach rate and avg cost per supplier.
``HAVING breach_rate > 20`` filters chronic offenders.

**Raw JSON Submission**

.. code-block:: json

   {
     "q_id": 12,
     "difficulty": "Medium",
     "db_type": "PostgreSQL",
     "domain": "Supply Chain",
     "instruction": "Which suppliers consistently deliver outside their agreed lead time windows, and what is the financial impact of late deliveries?",
     "context": "Procurement team quarterly supplier review. High lead time variance causes stock-outs and emergency procurement costs. SLA breach rate drives contract decisions.",
     "metrics_and_aggregation": [
       {"kpi_metric_name": "Actual Lead Time (days)", "aggregation_formula": "DATEDIFF(actual_delivery_date, order_date) per purchase_order_id"},
       {"kpi_metric_name": "Lead Time Variance (days)", "aggregation_formula": "actual_lead_time_days - dim_supplier.agreed_lead_time_days"},
       {"kpi_metric_name": "SLA Breach Rate (%)", "aggregation_formula": "ROUND(100.0 * COUNT(CASE WHEN actual > agreed THEN 1 END) / COUNT(*), 2) per supplier_id"},
       {"kpi_metric_name": "Avg Late Delivery Cost", "aggregation_formula": "AVG(emergency_procurement_cost) WHERE is_late = TRUE, per supplier_id"}
     ],
     "chain_of_thought": [
       "Step 1: Find suppliers with >20% SLA breach rate and quantify financial impact.",
       "Step 2: Need fact_purchase_orders, dim_supplier, dim_product, dim_date.",
       "Step 3: Compute actual_lead_time = DATEDIFF(actual_delivery_date, order_date) per PO.",
       "Step 4: Compare vs dim_supplier.agreed_lead_time_days. Flag late = actual > agreed.",
       "Step 5: Aggregate per supplier — breach_rate, avg_variance, avg_late_cost.",
       "Step 6: HAVING breach_rate > 20. Order by avg_late_cost DESC — highest financial impact first."
     ],
     "schema_tables": {
       "fact_tables": ["fact_purchase_orders"],
       "dimension_tables": ["dim_supplier", "dim_product", "dim_date"]
     },
     "data_model_layers": {
       "hierarchies": "Supplier > Category > Region, Date > Month > Quarter",
       "aggregations": "agg_supplier_lead_time_quarterly, agg_sla_breach_rate",
       "snapshots": "snap_supplier_performance_monthly"
     },
     "sql": "WITH po_lead_times AS (SELECT po.purchase_order_id, po.supplier_id, s.supplier_name, s.agreed_lead_time_days, DATEDIFF(po.actual_delivery_date, po.order_date) AS actual_lead_time_days, DATEDIFF(po.actual_delivery_date, po.order_date) - s.agreed_lead_time_days AS lead_time_variance, CASE WHEN DATEDIFF(po.actual_delivery_date, po.order_date) > s.agreed_lead_time_days THEN 1 ELSE 0 END AS is_late, po.emergency_procurement_cost FROM fact_purchase_orders po JOIN dim_supplier s ON po.supplier_id = s.supplier_id JOIN dim_date d ON po.date_id = d.date_id WHERE d.full_date >= CURRENT_DATE - INTERVAL '90 days') SELECT supplier_name, agreed_lead_time_days, COUNT(purchase_order_id) AS total_orders, SUM(is_late) AS late_orders, ROUND(100.0 * SUM(is_late) / COUNT(purchase_order_id), 2) AS sla_breach_rate_pct, ROUND(AVG(lead_time_variance), 1) AS avg_lead_time_variance_days, ROUND(AVG(CASE WHEN is_late = 1 THEN emergency_procurement_cost END), 2) AS avg_late_delivery_cost FROM po_lead_times GROUP BY supplier_id, supplier_name, agreed_lead_time_days HAVING ROUND(100.0 * SUM(is_late) / COUNT(purchase_order_id), 2) > 20 ORDER BY avg_late_delivery_cost DESC;"
   }
