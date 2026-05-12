Schema Reference
================

Standard retail schema used across all contributions.
Use these exact table and column names in your submissions.

Tables
------

.. list-table::
   :header-rows: 1
   :widths: 20 15 65

   * - Table
     - Type
     - Key Columns
   * - ``fact_sales``
     - Fact
     - ``sale_id``, ``customer_id``, ``product_id``, ``store_id``, ``date_id``, ``net_sales``, ``quantity``, ``is_return``
   * - ``dim_product``
     - Dimension
     - ``product_id``, ``product_name``, ``category``, ``sub_category``, ``brand``, ``unit_cost``
   * - ``dim_customer``
     - Dimension
     - ``customer_id``, ``customer_segment``, ``acquisition_channel``, ``region``
   * - ``dim_store``
     - Dimension
     - ``store_id``, ``store_name``, ``city``, ``region``, ``opening_date``
   * - ``dim_date``
     - Dimension
     - ``date_id``, ``full_date``, ``week``, ``month``, ``quarter``, ``year``

Common Join Patterns
---------------------

.. code-block:: sql

   -- Sales with product and date
   FROM fact_sales s
   JOIN dim_product p ON s.product_id = p.product_id
   JOIN dim_date d    ON s.date_id    = d.date_id

   -- Sales with store
   FROM fact_sales s
   JOIN dim_store st  ON s.store_id = st.store_id

   -- Full star join
   FROM fact_sales s
   JOIN dim_product  p  ON s.product_id  = p.product_id
   JOIN dim_customer c  ON s.customer_id = c.customer_id
   JOIN dim_store    st ON s.store_id    = st.store_id
   JOIN dim_date     d  ON s.date_id     = d.date_id

Common Filters
---------------

.. code-block:: sql

   -- Exclude returns
   WHERE s.is_return = FALSE

   -- Rolling 12-month window
   AND d.full_date >= CURRENT_DATE - INTERVAL '12 months'

   -- Rolling 90-day window
   AND d.full_date >= CURRENT_DATE - INTERVAL '90 days'

   -- Specific quarter and year
   AND d.year = 2024 AND d.quarter = 4

   -- Exclude newly opened stores
   AND st.opening_date <= CURRENT_DATE - INTERVAL '90 days'
