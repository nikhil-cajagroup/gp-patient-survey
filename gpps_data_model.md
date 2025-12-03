
# GP Patient Survey (GPPS) Data Model

This document describes how the **GP Patient Survey (GPPS)** data has been loaded into the `test-gp-patient-survey` database and how it relates to the official GPPS outputs. It is structured for both **human readability** and **programmatic use** (e.g. with LangChain).

---

## 1. GP Patient Survey in brief

The **GP Patient Survey** is an annual survey of patients registered with GP practices in England. It asks about:

- Getting through to the practice (phone, online, in person)
- Booking and attending appointments
- Continuity of care
- Out-of-hours services
- Overall experience of the practice

The public datasets are **aggregated** (not row-per-person). Each record represents a combination of:

- A **geography** (e.g. practice, PCN, ICS, region, England)
- A **survey year**
- A **question / variable**
- A **response category** or **summary measure**

For each such combination the data provides:

- Unweighted response counts
- Weighted percentages
- Confidence intervals
- Denominators / sample sizes
- Flags for suppression / low reliability

---

## 2. High‑level structure of the GPPS source data

From the official CSV/Excel releases and documentation, the key concepts are:

1. **Questionnaire and variables**
   - Each question has a **code** (e.g. `Q1`, `Q2`…).
   - Each question has multiple **response options** (e.g. *Very good*, *Fairly good*).
   - There are **derived variables** (e.g. “overall experience – good”) that pool categories.

2. **Geographical hierarchy**
   - England (national)
   - NHS Region
   - Integrated Care System (ICS/ICB)
   - Primary Care Network (PCN)
   - GP practice
   - Legacy: Clinical Commissioning Group (CCG)

3. **Outputs**
   - Separate files for each geography level.
   - Each file contains rows for question/variable × response category with statistics in columns.

Your database schema mirrors this structure in a relational form.

---

## 3. Database schema overview

Database: `test-gp-patient-survey`  
Partitioned tables:

- `gpps_variables` – **data dictionary** of questions, variables and response categories.
- `gpps_national` – England‑level results.
- `gpps_region` – NHS Region‑level results.
- `gpps_ics` – ICS/ICB‑level results.
- `gpps_ccg` – legacy CCG‑level results (historic).
- `gpps_pcn` – PCN‑level results.
- `gpps_practice` – GP practice‑level results.

Conceptually, each results table contains:

- Identifiers for **year**, **geography**, and **variable**
- Measures: **unweighted counts**, **weighted percentages**, **confidence intervals**, **denominators**
- Various **quality flags** (suppression, low reliability, etc.)

---

## 4. Table‑by‑table description

### 4.1 `gpps_variables` – questionnaire & variable catalogue

This is the **central reference table** for all GPPS questions and derived measures.

Typical columns (names may vary):

- `variable_id` or `question_code`  
  - Short code used across all results tables (e.g. `Q18_OVERALL_EXP`).
- `question_text`  
  - Full wording as seen on the questionnaire.
- `theme` / `topic`  
  - Logical grouping, e.g. *Access*, *Making an appointment*, *Experience of your GP practice*.
- `response_option_code`  
  - Code for each response option (e.g. `VERY_GOOD`, `FAIRLY_GOOD`).
- `response_option_label`  
  - Human‑readable label.
- `summary_group`  
  - Indicates membership of summary measures (e.g. “good experience” combines *Very good* + *Fairly good*).
- `is_derived`, `is_core` (boolean flags)  
  - Distinguish derived variables and core questionnaire items.
- `first_year`, `last_year`  
  - Range of survey years for which this variable is valid.

**Usage**

- All results tables store a compact `variable_id`.
- To show friendly labels in dashboards or reports, you `JOIN` to `gpps_variables`.
- This provides a machine‑readable replacement for the GPPS technical annex and questionnaire PDFs.

---

### 4.2 `gpps_national` – England‑level results

Each row represents:

> `survey_year × variable_id × response_option` (or summary) for **England** as a whole.

Common columns:

- `survey_year`
- `variable_id` (FK to `gpps_variables`)
- `measure_type` or `value_type` (e.g. `"response_option"`, `"summary"`)
- `unweighted_n` – number of respondents in that category
- `weighted_percent` – weighted percentage (headline figure)
- `ci_lower`, `ci_upper` – 95% confidence interval
- `denominator` – total valid responses for that question
- Quality flags – e.g. `is_suppressed`, `is_low_reliability`

This table reproduces the **England** tab in the official outputs.

---

### 4.3 `gpps_region` – NHS Region‑level results

Each row represents:

> `survey_year × region × variable_id × response_option`

Typical columns:

- `survey_year`
- `region_code`
- `region_name`
- `variable_id`
- Standard measures: `unweighted_n`, `weighted_percent`, `ci_lower`, `ci_upper`, `denominator`, flags.

This aligns with region‑level tables and lets you compare regional variation across England.

---

### 4.4 `gpps_ics` – Integrated Care System‑level results

Each row represents:

> `survey_year × ics × variable_id × response_option`

Columns:

- `survey_year`
- `ics_code`
- `ics_name`
- Optional: `region_code`, `region_name` for hierarchy
- Measures: `unweighted_n`, `weighted_percent`, `ci_lower`, `ci_upper`, `denominator`, flags.

This corresponds to the modern mid‑tier commissioning geography and is key for system‑level monitoring and benchmarking.

---

### 4.5 `gpps_ccg` – legacy CCG‑level results

Older survey years are still published at CCG level. This table preserves that history.

Each row represents:

> `survey_year × ccg × variable_id × response_option`

Columns:

- `survey_year`
- `ccg_code`
- `ccg_name`
- Optional mappings to `ics_code` / `region_code` for cross‑walks.
- The standard GPPS measures and flags.

Use this when you need **time series** that pre‑date ICSs.

---

### 4.6 `gpps_pcn` – Primary Care Network‑level results

Each row represents:

> `survey_year × pcn × variable_id × response_option`

Columns:

- `survey_year`
- `pcn_code`
- `pcn_name`
- Optional: `ics_code`, `region_code`
- Measures + flags as before.

This table is ideal for:

- Comparing practices within a PCN.
- Comparing PCNs within an ICS or region.
- Identifying variation between PCNs.

---

### 4.7 `gpps_practice` – GP practice‑level results

Usually the **largest table**.

Each row represents:

> `survey_year × practice × variable_id × response_option`

Columns typically include:

- `survey_year`
- `practice_code` (ODS code)
- `practice_name`
- Hierarchy: `pcn_code`, `ics_code`, `region_code`
- Measures: `unweighted_n`, `weighted_percent`, `ci_lower`, `ci_upper`, `denominator`
- Quality flags: suppression, reliability, sample size bands
- Optional: practice characteristics from other sources (list size, rural/urban, deprivation quintile, etc.)

Use this table when:

- Benchmarking a practice against its **PCN / ICS / region / national** results.
- Building **practice‑level dashboards** and time series.
- Analysing distributions of scores across all practices.

---

## 5. Partitioning strategy

All results tables are **partitioned**, typically by `survey_year` (and optionally by geography code prefixes).

Benefits:

- Queries that filter on `survey_year` only read the relevant partitions.
- New survey years can be appended as new partitions.
- Longitudinal analysis is still straightforward by scanning multiple years.

This design is well‑suited to Athena/Presto/Trino and other columnar/“data lake” environments.

---

## 6. Example query patterns (SQL snippets)

Below are conceptual SQL examples your LLM / LangChain pipeline can adapt.

### 6.1 Lookup a question’s wording and theme

```sql
SELECT
  variable_id,
  question_text,
  theme,
  response_option_label
FROM gpps_variables
WHERE variable_id = 'Q18_OVERALL_EXP';
```

### 6.2 Practice vs ICS vs England for one question

```sql
-- Practice
SELECT 'practice' AS level, survey_year, weighted_percent
FROM gpps_practice
WHERE practice_code = 'A12345'
  AND variable_id = 'Q18_OVERALL_EXP'
  AND response_option_code = 'GOOD';

-- ICS
SELECT 'ics' AS level, survey_year, weighted_percent
FROM gpps_ics
WHERE ics_code = '01A'
  AND variable_id = 'Q18_OVERALL_EXP'
  AND response_option_code = 'GOOD';

-- England
SELECT 'national' AS level, survey_year, weighted_percent
FROM gpps_national
WHERE variable_id = 'Q18_OVERALL_EXP'
  AND response_option_code = 'GOOD';
```

Your orchestration layer can union these results and present them as a single comparison chart or table.

### 6.3 Time series for all practices in an ICS (one variable)

```sql
SELECT
  survey_year,
  practice_code,
  weighted_percent
FROM gpps_practice
WHERE ics_code = '01A'
  AND variable_id = 'Q18_OVERALL_EXP'
  AND response_option_code = 'GOOD';
```

---

## 7. How this supports LangChain / LLM use

This schema is LLM‑friendly because:

- **Consistent column names** across tables make it easy for an agent to construct joins and comparisons.
- `gpps_variables` gives a **single source of truth** for question wording, themes and response labels.
- Separate tables per geography map neatly onto natural‑language prompts like:
  - “Compare this practice to its ICS and to England.”
  - “Show me regional variation for access questions in 2023.”
- Partitioning by `survey_year` keeps scans efficient even when the model generates broad queries.

When wiring this up via LangChain:

- Define one **SQLDatabase** (or equivalent) that points at `test-gp-patient-survey`.
- Provide this document as **system or tool documentation**, so the agent understands:
  - Which tables exist.
  - How they relate to each other.
  - Which columns to use for common analytical tasks.

This should make your GPPS data both **human‑navigable** and **AI‑navigable**.
