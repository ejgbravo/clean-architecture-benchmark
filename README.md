# Clean Architecture Benchmark

A curated dataset of open source **ERP/CRM** projects for the automatic evaluation of **Clean Architecture** compliance using **Large Language Models (LLMs)**.

---

## Purpose

This benchmark enables:

1. **Reproducible evaluation** — a fixed set of real-world projects with expert-annotated ground truth scores.
2. **LLM calibration** — reference prompts and few-shot examples so that LLMs score projects consistently.
3. **Architecture research** — a structured comparison of how different frameworks and ecosystems approach (or ignore) Clean Architecture.

---

## Dataset Contents

```
clean-architecture-benchmark/
├── data/
│   ├── projects.json        # Curated list of ERP/CRM projects with metadata
│   └── ground_truth.json    # Expert ground-truth Clean Architecture scores
├── criteria/
│   ├── clean_architecture.json  # Machine-readable evaluation criteria and scoring rubric
│   └── rubric.md                # Human-readable evaluation rubric
└── prompts/
    └── evaluation_prompt.md     # LLM prompt template with few-shot examples
```

---

## Projects

The dataset covers **10 open source ERP/CRM systems** spanning Python, PHP, and Java:

| Project | Type | Language | Score | Level |
|---------|------|----------|-------|-------|
| [metasfresh](https://github.com/metasfresh/metasfresh) | ERP | Java | 16/18 | 🏆 Full |
| [OroCRM](https://github.com/oroinc/crm) | CRM | PHP (Symfony) | 14/18 | ✅ Mostly |
| [EspoCRM](https://github.com/espocrm/espocrm) | CRM | PHP | 11/18 | ⚠️ Partial |
| [Tryton](https://github.com/tryton/tryton) | ERP | Python | 9/18 | ⚠️ Partial |
| [iDempiere](https://github.com/idempiere/idempiere) | ERP | Java (OSGi) | 9/18 | ⚠️ Partial |
| [Odoo](https://github.com/odoo/odoo) | ERP+CRM | Python | 5/18 | ❌ None |
| [ERPNext](https://github.com/frappe/erpnext) | ERP | Python | 5/18 | ❌ None |
| [Vtiger CRM](https://github.com/vtiger-crm/vtiger) | CRM | PHP | 4/18 | ❌ None |
| [SuiteCRM](https://github.com/salesagility/SuiteCRM) | CRM | PHP | 3/18 | ❌ None |
| [Dolibarr](https://github.com/Dolibarr/dolibarr) | ERP+CRM | PHP | 1/18 | ❌ None |

---

## Evaluation Criteria

Each project is scored on **6 criteria** (0–3 each, max 18):

| # | Criterion | What is measured |
|---|-----------|-----------------|
| 1 | **Layer Separation** | Are the four Clean Architecture layers clearly separated? |
| 2 | **Dependency Rule** | Do dependencies always point inward? |
| 3 | **Entity Independence** | Are domain entities free from framework/ORM dependencies? |
| 4 | **Use Case Layer** | Is there a clear, framework-free application service layer? |
| 5 | **Testability** | Can domain logic be unit-tested without infrastructure? |
| 6 | **Abstraction Usage** | Are interfaces used for all external dependencies? |

See [`criteria/rubric.md`](criteria/rubric.md) for the full rubric and [`criteria/clean_architecture.json`](criteria/clean_architecture.json) for the machine-readable version.

---

## Compliance Levels

| Score | Level | Description |
|-------|-------|-------------|
| 0–5 | ❌ None | No meaningful Clean Architecture adherence |
| 6–11 | ⚠️ Partial | Some elements present (service layers, repositories), but inconsistently applied |
| 12–15 | ✅ Mostly | Good adherence with minor violations |
| 16–18 | 🏆 Full | Consistent application of all Clean Architecture principles |

---

## Using the Dataset for LLM Evaluation

### 1. Select a project from `data/projects.json`

Each project entry includes:
- Repository URL and key directory paths
- Notable architectural characteristics
- Representative source file paths to fetch and include in the prompt

### 2. Build the evaluation prompt

Use the template in [`prompts/evaluation_prompt.md`](prompts/evaluation_prompt.md):
- Fill in project metadata
- Include 3 representative code samples (entity, service, repository)
- Optionally add few-shot examples for calibration

### 3. Submit to an LLM and collect JSON output

The prompt requests a structured JSON response with per-criterion scores, justifications, and an overall assessment.

### 4. Compare to ground truth

Compare the LLM's scores against `data/ground_truth.json` to measure:
- **Exact match accuracy** — percentage of criteria where the LLM score equals ground truth
- **Mean Absolute Error (MAE)** — average absolute difference between LLM and ground truth scores
- **Compliance level accuracy** — percentage of projects where the LLM classifies the compliance level correctly

---

## Clean Architecture Reference

This dataset is based on the principles defined in:

> Martin, Robert C. *Clean Architecture: A Craftsman's Guide to Software Structure and Design*. Prentice Hall, 2017. ISBN 978-0134494166.

Key principles evaluated:
- **The Dependency Rule**: Source code dependencies can only point inward.
- **Entities** encapsulate enterprise business rules and are framework-independent.
- **Use Cases** encapsulate application-specific business rules.
- **Interface Adapters** translate data between use cases and external agencies.
- **Frameworks and Drivers** are the outermost layer (databases, web, UI).

---

## License

The dataset annotations and evaluation materials in this repository are released under the [MIT License](LICENSE).

The open source projects listed in the dataset retain their own licenses (LGPL-3.0, GPL-3.0, AGPL-3.0, etc.). This repository does not redistribute any of their source code.