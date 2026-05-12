Getting Started
===============

Who Should Contribute
---------------------

- Data Analysts writing SQL against retail data warehouses
- BI Engineers building dashboards and reports
- Data Scientists working with retail transaction data
- Retail Domain Experts who understand the business problems
- Anyone who has turned a business question into a SQL query

Quick Start (UI Form — Recommended)
------------------------------------

1. Open the `Query Entry Form <https://infometry-infofiscus.github.io/text2sql_data_collection_platform/>`_
2. Fill all required fields — Live JSON panel updates in real time
3. Copy generated JSON from the Live JSON panel
4. Save as ``.json`` file following the :ref:`naming convention <naming>`
5. Place in ``submissions/pending/`` and open a Pull Request

Quick Start (Manual JSON)
--------------------------

.. code-block:: bash

   git clone https://github.com/Infometry-Infofiscus/retail-analytics-questions-sql.git
   cd retail-analytics-questions-sql
   cp templates/submission_template.json submissions/pending/retail_<topic>_<desc>.json

Fill all required fields per :doc:`data_spec`, then submit a Pull Request.

.. _naming:

File Naming Convention
-----------------------

.. code-block:: text

   <domain>_<topic>_<short_description>.json

Rules:

- Lowercase only
- Words separated by underscores
- Start with domain prefix (``retail_``, ``finance_``, ``healthcare_``)
- Under 60 characters total

Examples::

   retail_revenue_category_quarterly_decline.json
   retail_inventory_slow_moving_products.json
   retail_customers_high_value_segmentation.json
