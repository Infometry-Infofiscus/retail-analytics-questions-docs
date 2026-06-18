Analytics Text-to-SQL Dataset
==============================

A community-driven, open dataset of real-world business analytics use cases
for training and benchmarking text-to-SQL systems — spanning Retail, Healthcare,
Finance, SaaS, Manufacturing, Supply Chain, and more.

Why This Dataset?
-----------------

Most text-to-SQL benchmarks rely on synthetic or academic data that doesn't
reflect how business users actually ask questions. Real analytics work involves
domain-specific terminology, multi-table star schema joins, KPI logic, and
context that only domain experts understand.

This dataset is different:

- **Real business questions** — written the way analysts and stakeholders ask them
- **Multi-domain** — Retail, Healthcare, Finance, SaaS, Manufacturing, Supply Chain
- **Structured submissions** — every entry includes business context, KPIs, chain of thought, and SQL
- **Community-driven** — built by practitioners, for practitioners
- **Benchmark-ready** — designed for training and evaluating text-to-SQL systems

Who Should Contribute?
-----------------------

- Data Analysts writing SQL against business warehouses
- BI Engineers building dashboards and reports
- Domain Experts who understand real business problems
- Data Scientists working on NLP and text-to-SQL research
- Anyone who has turned a business question into a SQL query

How to Contribute
-----------------

No repository access needed. All submissions go through the UI form:

1. Open the `Query Entry Form <https://infometry-infofiscus.github.io/InfoFiscus-Stratum/>`_
2. Fill in all required fields (business question, context, KPIs, schema, SQL)
3. Copy the generated JSON and post it in a `Discussion thread <https://github.com/Infometry-Infofiscus/retail-analytics-questions-docs/discussions>`_
4. Maintainers review and ingest — target turnaround: 7 days

Full documentation: https://retail-analytics-docs.readthedocs.io/en/latest/

Supported Domains
-----------------

``Retail`` · ``Healthcare`` · ``HighTech (SaaS)`` · ``Finance`` · ``Manufacturing`` · ``Supply Chain`` · ``Other``

Supported Databases
-------------------

``BigQuery`` · ``Snowflake`` · ``Redshift`` · ``PostgreSQL`` · ``MySQL`` · ``Oracle`` · ``Azure Synapse`` · ``Other``

License
-------

Open dataset. Contributions are accepted under community guidelines.
See `Contributing Guide <https://retail-analytics-docs.readthedocs.io/en/latest/contributing.html>`_ for details.
