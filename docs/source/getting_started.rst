Getting Started
===============

Who Should Contribute
---------------------

- Data Analysts writing SQL against retail data warehouses
- BI Engineers building dashboards and reports
- Data Scientists working with retail transaction data
- Retail Domain Experts who understand the business problems
- Anyone who has turned a business question into a SQL query

How to Submit
-------------

All contributions go through the UI Form. The dataset repository is managed
privately by maintainers — contributors submit via the form and maintainers
handle ingestion.

Step 1 — Open the Form
~~~~~~~~~~~~~~~~~~~~~~~

Open the `Query Entry Form <https://infometry-infofiscus.github.io/text2sql_data_collection_platform/>`_
in your browser.

Step 2 — Fill Required Fields
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Complete all required sections. The **Live JSON panel** on the right
updates in real time as you type. See :doc:`data_spec` for full field reference.

Step 3 — Copy and Send
~~~~~~~~~~~~~~~~~~~~~~~

Copy the generated JSON from the Live JSON panel.
Send it via the `Discussions page <https://github.com/Infometry-Infofiscus/retail-analytics-questions-docs/discussions>`_
or contact a maintainer directly.

.. _naming:

File Naming Convention
-----------------------

Maintainers will name files on ingest, but use this pattern when sharing:

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
