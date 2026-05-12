Data Spec
=========

Full field specification for all submission entries.

Required vs Optional
--------------------

.. list-table::
   :header-rows: 1
   :widths: 10 30 15

   * - Section
     - UI Label
     - Required
   * - 1
     - Meta (difficulty, db_type, domain)
     - ✅ Yes
   * - 2
     - Business Question
     - ✅ Yes
   * - 3
     - Business Context
     - ✅ Yes
   * - 4
     - Metrics & Aggregation
     - ✅ Yes
   * - 5
     - Chain of Thought
     - ⬜ Optional
   * - 6
     - Schema Tables
     - ✅ Yes
   * - 7
     - Data Model Layers
     - ✅ Yes
   * - 8
     - SQL Answer
     - ⬜ Optional

Field Reference
---------------

.. list-table::
   :header-rows: 1
   :widths: 25 15 15 10 35

   * - Field
     - UI Control
     - Data Type
     - Required
     - Validation
   * - ``difficulty``
     - Dropdown
     - string
     - Yes
     - Easy / Medium / Hard / Expert
   * - ``db_type``
     - Dropdown
     - string
     - Yes
     - BigQuery / Snowflake / Redshift / PostgreSQL / MySQL / Oracle / Azure Synapse / Other
   * - ``domain``
     - Dropdown
     - string
     - Yes
     - Retail / Healthcare / HighTech (SaaS) / Finance / Manufacturing / Supply Chain / Other
   * - ``instruction``
     - Textarea
     - string
     - Yes
     - Non-empty
   * - ``context``
     - Textarea
     - string
     - Yes
     - Non-empty
   * - ``required_metrics_kpis[]``
     - Dynamic rows
     - array[string]
     - Yes
     - At least one KPI name
   * - ``aggregation_logic{}``
     - Dynamic rows
     - object
     - Yes
     - Each KPI must have paired formula
   * - ``chain_of_thought[]``
     - Dynamic list
     - array[string]
     - No
     - Preserve order if provided
   * - ``data_model.facts``
     - Dynamic list
     - string
     - Yes
     - Comma-separated, at least one
   * - ``data_model.dims``
     - Dynamic list
     - string
     - Yes
     - Comma-separated, at least one
   * - ``data_model.hierarchies``
     - Text input
     - string
     - Yes
     - Free text
   * - ``data_model.aggrs``
     - Text input
     - string
     - Yes
     - Free text
   * - ``data_model.snapshots``
     - Text input
     - string
     - Yes
     - Free text (can be blank)
   * - ``sql``
     - SQL textarea
     - string
     - No
     - Free text SQL

Canonical Payload Shape
------------------------

.. code-block:: json

   {
     "q_id": 1,
     "difficulty": "Medium",
     "db_type": "Relational (SQL)",
     "domain": "Retail",
     "instruction": "Which product categories generated the most revenue last quarter?",
     "context": "Merchandising leadership preparing for annual planning meeting.",
     "metrics_and_aggregation": [
       {
         "kpi_metric_name": "Category Revenue (Current Quarter)",
         "aggregation_formula": "SUM(net_sales) WHERE year=2024 AND quarter=4, grouped by category"
       }
     ],
     "chain_of_thought": [
       "Step 1: Goal is to rank product categories by net revenue for Q4."
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
     "sql": "SELECT p.category, SUM(s.net_sales) FROM fact_sales s JOIN dim_product p ON s.product_id = p.product_id GROUP BY p.category;"
   }

Transformation Rules
---------------------

1. ``required_metrics_kpis`` — flat array of KPI name strings (trimmed)
2. ``aggregation_logic`` — key-value object: key = metric name, value = formula
3. Fact table list → ``data_model.facts`` as comma-separated string
4. Dim table list → ``data_model.dims`` as comma-separated string
5. ``chain_of_thought`` — submit as ``[]`` if empty
6. ``sql`` — submit as ``""`` if empty
7. Preserve all ``data_model`` keys even if blank

Validation Rules
-----------------

**Fails when:**

- Any of ``difficulty``, ``db_type``, ``domain`` missing
- ``instruction`` blank
- ``context`` blank
- No valid metric + formula pair
- No fact table in ``data_model.facts``
- No dimension table in ``data_model.dims``

**Passes when:**

- ``chain_of_thought`` is ``[]``
- ``data_model.hierarchies``, ``data_model.aggrs``, ``data_model.snapshots`` are blank strings
- ``sql`` is blank
