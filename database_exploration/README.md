# NYC High School Analysis  
## Enrollment, English Language Learners, and Special Education

---

## Overview

This document describes **one analytical component** of a broader NYC schools data project.

The purpose of this module is to analyze New York City high schools with respect to:

- School distribution across boroughs
- English Language Learners (ELL)
- Special Education (SPED) student representation

The **primary technical focus** of this module is **data extraction, filtering, aggregation, and merging performed directly in SQL**.  
Python is used mainly as an orchestration and validation layer.

---

## Context Within the Overall Project

This analysis is **not a standalone project**, but part of a larger end-to-end data pipeline that includes:

- Data ingestion into a PostgreSQL database
- SQL-based transformations and analytical queries
- Downstream analysis and visualization modules

This module specifically serves as a **SQL-centric analytical layer**, producing clean, aggregated results at the borough and school level.

---

## Technical Stack

- **Database:** PostgreSQL (Neon)
- **Query Language:** SQL (primary transformation layer)
- **Programming Language:** Python
- **Libraries:**
  - `pandas` – query execution, light transformations, validation
  - `psycopg2` – PostgreSQL database connection

---

## Analytical Approach

All major data preparation steps are implemented **inside SQL queries**, including:

- Filtering
- Aggregation
- Ranking
- Joins across multiple tables

Python is intentionally limited to:

- Executing SQL queries
- Inspecting query outputs
- Validating SQL results using pandas operations

This design ensures:

- Minimal data movement
- Reproducible transformations
- Clear separation between data extraction and analysis logic
- High transparency for debugging and review

---

## Database Connection

A direct PostgreSQL connection is established using `psycopg2`.

> **Note:**  
> Hardcoded credentials are used here **for onboarding and learning purposes only**.  
> In production, credentials should be stored securely (e.g., environment variables or secrets managers).

---

## Analysis Steps

### 1. School Distribution per Borough

**Objective:**  
Determine how many distinct high schools exist in each NYC borough.

**Method:**

- Query the `high_school_directory` table
- Count distinct school identifiers (`dbn`)
- Group results by borough

**Result Summary:**

| Borough        | Schools |
|---------------|---------|
| Brooklyn       | 121 |
| Bronx          | 118 |
| Manhattan      | 106 |
| Queens         | 80 |
| Staten Island  | 10 |

Brooklyn contains the largest number of high schools, while Staten Island has the fewest.

---

### 2. Average Percentage of English Language Learners (ELL)

**Objective:**  
Compute the average share of English Language Learners per borough.

**Method:**

- LEFT JOIN between:
  - `high_school_directory`
  - `school_demographics`
- Aggregation using `AVG(ell_percent)`
- Grouped by borough

**Result Summary:**

| Borough        | Avg ELL % |
|---------------|-----------|
| Manhattan      | 7.57 |
| Bronx          | NaN |
| Brooklyn       | NaN |
| Queens         | NaN |
| Staten Island  | NaN |

**Interpretation:**

- Only Manhattan returned a numeric ELL average.
- All other boroughs returned `NaN`, despite valid school identifiers.
- This strongly suggests **incomplete or missing ELL demographic data** for those boroughs.

---

### 3. Top 3 Schools per Borough by Special Education (SPED) Percentage

**Objective:**  
Identify the top 3 schools per borough with the highest percentage of students in special education programs.

**Method:**

- Use a Common Table Expression (CTE) to:
  - Select the maximum `sped_percent` per school
  - Exclude null values
- Rank schools within each borough using `ROW_NUMBER()`
- Filter to the top 3 per borough

**Result Summary (Observed):**

| Borough   | DBN    | School Name                                      | SPED % |
|----------|--------|--------------------------------------------------|--------|
| Manhattan | 01M450 | East Side Community School                       | 28.8 |
| Manhattan | 01M509 | Marta Valle High School                          | 25.9 |
| Manhattan | 01M292 | Henry Street School for International Studies    | 25.1 |

Only Manhattan produced valid results, again pointing to **data coverage limitations** in the demographic table.

---

## Python-Based Verification

Although SQL is the primary transformation layer, an additional verification step is performed in Python to:

- Confirm join logic
- Validate ranking results
- Identify issues caused by missing demographic data

**Verification Steps:**

1. Load both tables fully into pandas
2. Standardize `dbn` formatting (trim + uppercase)
3. Perform a left join in Python
4. Recompute top 3 SPED schools per borough

**Outcome:**

- Python results confirmed SQL behavior
- Missing SPED values for most boroughs persisted
- Confirms that observed gaps originate from the source data, not query logic

---

## Key Insights

1. **School Distribution**
   - Brooklyn has the highest number of high schools
   - Staten Island has very few, but typically larger schools

2. **Data Completeness Issues**
   - ELL and SPED data is largely missing for Bronx, Brooklyn, Queens, and Staten Island
   - Manhattan has the most complete demographic coverage

3. **Special Education Concentration**
   - The highest SPED percentages are observed in Manhattan schools
   - Other boroughs cannot be reliably compared due to missing data

---

## Limitations

- Incomplete demographic data limits borough-level comparisons
- Results for ELL and SPED should not be interpreted as true absence, but as **missing coverage**
- Further data validation is required before production use

---

## Next Steps

- Investigate missing records in `school_demographics`
- Validate data ingestion logic for demographic tables
- Confirm that all schools in `high_school_directory` are represented
- Re-run aggregations after data coverage is improved

---

## Role in the Overall Project

This module functions as a **SQL-first analytical extraction layer**, providing:

- Validated borough-level metrics
- School-level rankings
- Clear diagnostics on data quality issues

Its outputs are intended to feed downstream analysis, reporting, and visualization components within the larger NYC schools data project.
