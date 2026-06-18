Getting Started
===============

.. note::
   📊 Open Dataset · SQL · Multi-Domain · Community Contributed

Help build the largest open Text-to-SQL dataset for real-world business analytics.
Every query you submit trains better AI for real business problems — across any domain.

----

Who Should Contribute
---------------------

- Data Analysts writing SQL against business data warehouses
- BI Engineers building dashboards and reports
- Data Scientists working on NLP and text-to-SQL research
- Domain Experts (Healthcare, Finance, SaaS, Manufacturing, Supply Chain)
- Anyone who has turned a business question into a SQL query

----

Supported Domains
-----------------

Submissions accepted across all domains:

.. list-table::
   :header-rows: 1
   :widths: 25 75

   * - Domain
     - Example Use Cases
   * - **Retail**
     - Sales revenue, inventory turnover, return rates, customer segmentation
   * - **Healthcare**
     - Readmission rates, claims denial, patient outcomes, utilization
   * - **HighTech (SaaS)**
     - Churn rate, feature adoption, ARR, funnel conversion, DAU/MAU
   * - **Finance**
     - Portfolio performance, risk exposure, transaction anomalies, alpha
   * - **Manufacturing**
     - OEE, downtime analysis, yield rate, defect tracking
   * - **Supply Chain**
     - Supplier lead time, SLA breach rate, stock-out frequency
   * - **Other**
     - Any domain with structured SQL data and real business questions

----

Supported Databases
-------------------

Submissions accepted for all major SQL engines:

``BigQuery`` · ``Snowflake`` · ``Redshift`` · ``PostgreSQL`` · ``MySQL`` · ``Oracle`` · ``Azure Synapse`` · ``Other``

Specify your ``db_type`` accurately — dialect differences (e.g. ``DATE_DIFF`` vs ``DATEDIFF``)
are expected and valuable for the dataset.

----

How to Submit *(~5 minutes per entry)*
---------------------------------------

All contributions go through the UI Form. The dataset repository is managed
privately by maintainers — contributors submit via the form and maintainers
handle ingestion.

Step 1 — Open the Form
~~~~~~~~~~~~~~~~~~~~~~~

Open the `Query Entry Form <https://infometry-infofiscus.github.io/InfoFiscus-Stratum/>`_
in your browser. No login or account needed.

Step 2 — Select Your Domain
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Choose your domain from the dropdown: Retail, Healthcare, HighTech (SaaS),
Finance, Manufacturing, Supply Chain, or Other.

Step 3 — Fill Required Fields
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Complete all required sections. The **Live JSON panel** on the right
updates in real time as you type. See :doc:`data_spec` for full field reference.

Required sections:

1. **Meta** — difficulty, db_type, domain
2. **Business Question** — how a real user would ask it
3. **Business Context** — who needs this and why
4. **Metrics & Aggregation** — KPI names + formulas
5. **Schema Tables** — fact and dimension tables used
6. **Data Model Layers** — hierarchies, aggregations, snapshots

Optional but strongly recommended:

7. **Chain of Thought** — step-by-step reasoning
8. **SQL Answer** — actual query (significantly improves quality)

Step 4 — Copy and Submit
~~~~~~~~~~~~~~~~~~~~~~~~~

Once all fields are complete, click the **Submit** button in the form.
Your entry will be sent directly to the maintainers for review.

.. tip::
   Best submissions have clear business context, realistic KPIs,
   and SQL that actually runs. See :doc:`examples` before writing your first entry.

----

What Makes a Good Submission
-----------------------------

**Strong submissions have:**

- A business question written the way a non-technical stakeholder would ask it
- Context that explains *who* needs this data and *what decision* it drives
- KPIs with clear, plain-English aggregation formulas (avoid SQL jargon in KPI names)
- SQL that runs cleanly against the standard schema or your specified db_type
- Chain of thought that walks through the reasoning step by step

**Weak submissions often have:**

- Vague instructions like "get sales data" with no business context
- KPI names that are just SQL expressions (``SUM(net_sales)``) instead of business terms
- Missing or mismatched metric/formula pairs
- SQL with syntax errors or non-standard table names without explanation

----

Difficulty Guide
----------------

Use this as a reference when selecting difficulty:

.. list-table::
   :header-rows: 1
   :widths: 15 25 60

   * - Level
     - Typical Pattern
     - Examples
   * - **Easy**
     - Single table, simple filter + aggregate
     - Total sales last month, top 10 products by revenue
   * - **Medium**
     - 2–3 table joins, date windows, basic window functions
     - YoY revenue comparison, customer segment breakdown, return rate
   * - **Hard**
     - Multi-CTE, advanced window functions, complex conditions
     - Seasonality index, OEE score, churn funnel, portfolio alpha
   * - **Expert**
     - Recursive CTEs, nested window functions, multi-step derivations
     - Cohort retention curves, graph traversal, multi-period attribution

----

Examples
--------

Click any example below to jump to the full walkthrough:

**Retail**

- :ref:`example-category-revenue-yoy`
- :ref:`example-slow-moving-inventory`

**Healthcare**

- :ref:`example-hospital-readmission-rate`
- :ref:`example-claims-denial-rate`

**HighTech (SaaS)**

- :ref:`example-saas-churn-rate`
- :ref:`example-saas-feature-adoption`

**Finance**

- :ref:`example-finance-portfolio-performance`

**Manufacturing**

- :ref:`example-manufacturing-oee`

**Supply Chain**

- :ref:`example-supplychain-supplier-lead-time`

----

FAQ
---

**Do I need to know SQL to contribute?**

SQL is optional but strongly recommended. Entries with SQL are higher quality
and more useful for model training. See :doc:`examples` for reference before writing.

**Can I submit from non-retail domains?**

Yes — all domains are welcome. Use equivalent fact/dimension table naming conventions
(e.g. ``fact_claims``, ``dim_patient``) following the star schema pattern in :doc:`schema_reference`.

**How many submissions can I make?**

No limit. Each unique business question counts as one entry. Bulk submissions
with diverse domains and difficulty levels are especially valued.

**What if my SQL has dialect-specific syntax?**

Specify your ``db_type`` correctly (e.g. BigQuery, Snowflake). Dialect-specific
functions like ``DATE_DIFF``, ``DATEADD``, ``FORMAT_DATE`` are expected and kept as-is.

**Can I submit without the SQL?**

Yes — SQL is optional. Entries without SQL are still accepted if all other
required fields are complete and high quality.

**What if my schema differs from the standard one?**

Note the variation in your submission. Non-standard tables are accepted
as long as they follow fact/dimension naming conventions.

**How long does review take?**

Target turnaround is 7 days. Complex or ambiguous entries may take longer.
You'll receive feedback directly in your Discussion thread.

----

.. seealso::
   :doc:`data_spec` · :doc:`schema_reference` · :doc:`examples` · `Submit Form <https://infometry-infofiscus.github.io/InfoFiscus-Stratum/>`_
