Contributing
============

Before You Begin
-----------------

- Read :doc:`data_spec` — required fields, validation rules, dropdown values
- Browse :doc:`examples` — see what good entries look like
- Base use case on real or realistic business scenario

Step-by-Step Guide
-------------------

Option A — UI Form (Recommended)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Open the `Query Entry Form <https://infometry-infofiscus.github.io/text2sql_data_collection_platform/>`_
2. Fill all required fields
3. Copy generated JSON from Live JSON panel
4. Save as ``.json`` following naming convention
5. Place in ``submissions/pending/`` and open a Pull Request

Option B — Manual JSON
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   git clone https://github.com/Infometry-Infofiscus/retail-analytics-questions-sql.git
   cp templates/submission_template.json submissions/pending/<domain>_<topic>_<desc>.json
   # fill fields, then:
   git add submissions/pending/your_file.json
   git commit -m "Add: retail <topic> example"
   git push origin main

Templates
----------

Two templates available in ``templates/``:

- ``submission_template.json`` — for developers comfortable with JSON
- ``submission_template.md`` — for anyone preferring plain text first

Contribution Guidelines
------------------------

**Do:**

- Use real-world or realistic business scenarios
- Write ``instruction`` as a business user would naturally ask it
- Write ``aggregation_logic`` in plain language — avoid SQL jargon
- Use standard schema table names
- Validate JSON at `jsonlint.com <https://jsonlint.com>`_ before submitting
- Include SQL if possible — significantly improves dataset quality

**Don't:**

- Include sensitive data, real customer names, or PII
- Reference internal systems or proprietary company names
- Use non-standard table names without noting the variation
- Submit incomplete entries with empty required fields
- Submit duplicates already in the examples folder

Review Process
---------------

1. Submit PR with file in ``submissions/pending/``
2. Maintainer reviews quality, correctness, completeness
3. Approved → moved to ``submissions/reviewed/`` and merged
4. Changes needed → feedback left on PR
5. Not suitable → moved to ``submissions/rejected/`` with reason

Target review time: **7 days**.

Quality Checklist
------------------

Before submitting verify:

- [ ] All required sections filled
- [ ] ``required_metrics_kpis`` list matches keys in ``aggregation_logic``
- [ ] ``data_model.facts`` and ``data_model.dims`` non-empty
- [ ] SQL (if provided) runs against standard schema
- [ ] File named following naming convention
- [ ] No proprietary, internal, or PII data included
