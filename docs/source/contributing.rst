Contributing
============

The dataset repository is privately managed by maintainers.
All contributions flow through the **UI Form** — no repository access needed.

Before You Begin
-----------------

- Read :doc:`data_spec` — required fields, validation rules, dropdown values
- Read :doc:`schema_reference` — standard table and column names
- Read :doc:`examples` — see what good entries look like
- Base your use case on a real or realistic business scenario from **any domain**

Submission Steps
-----------------

Step 1 — Open the UI Form
~~~~~~~~~~~~~~~~~~~~~~~~~~

Open the `Query Entry Form <https://infometry-infofiscus.github.io/InfoFiscus-Stratum/>`_
in your browser. No login or account needed.

Step 2 — Fill All Required Fields
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Complete all 8 sections. The **Live JSON panel** updates in real time.
Required sections: Meta, Business Question, Business Context,
Metrics & Aggregation, Schema Tables, Data Model Layers.
Optional: Chain of Thought, SQL Answer.

Step 3 — Copy the JSON
~~~~~~~~~~~~~~~~~~~~~~~~

Copy the full JSON output from the Live JSON panel on the right side of the form.

Step 4 — Submit to Maintainers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Paste your JSON in a new `Discussion thread <https://github.com/Infometry-Infofiscus/retail-analytics-questions-docs/discussions>`_
with the title format::

   [Submission] <domain>_<topic>_<short_description>

Maintainers will review and ingest into the dataset.

Contribution Guidelines
------------------------

**Do:**

- Use real-world or realistic business scenarios from any supported domain
- Write ``instruction`` as a business user would naturally ask it
- Write ``aggregation_logic`` in plain language — avoid SQL jargon
- Use standard schema table names from :doc:`schema_reference`
- Include SQL if possible — significantly improves dataset quality
- Make sure ``required_metrics_kpis`` list matches keys in ``aggregation_logic``

**Don't:**

- Include sensitive data, real customer names, or PII
- Reference internal systems or proprietary company names
- Use non-standard table names without noting the variation
- Submit incomplete entries with empty required fields
- Submit duplicates already covered in :doc:`examples`

Review Process
---------------

1. You post JSON in a Discussion thread
2. Maintainer reviews for quality, correctness, completeness
3. Approved → ingested into dataset, thread marked resolved
4. Changes needed → feedback left in the thread
5. Not suitable → declined with reason

Target review time: **7 days**.

Quality Checklist
------------------

Before submitting verify:

- [ ] All required sections filled
- [ ] ``required_metrics_kpis`` matches keys in ``aggregation_logic``
- [ ] ``data_model.facts`` and ``data_model.dims`` non-empty
- [ ] SQL (if provided) runs against standard schema without errors
- [ ] No proprietary, internal, or PII data included
