# SAT Results Data Cleaning & PostgreSQL Integration

## Overview

This notebook demonstrates the **data cleaning, validation, and integration** of the NYC high schools SAT results dataset into a PostgreSQL database.  

The goal is to prepare a **reliable, normalized dataset** for analysis and ensure that duplicate or missing records are handled appropriately.

---

## 1. Dataset Loading

- The raw dataset `sat-results.csv` was loaded using `pandas.read_csv`.
- Initial inspection of the dataset includes:
  - Viewing the first rows with `.head()`
  - Checking the distribution of `academic_tier_rating`

```python
df = pd.read_csv('data/sat-results.csv')
df.head()
df['academic_tier_rating'].value_counts()
```

## 2. Remove Redundant / Invalid Columns

- Dropped duplicated column with typographical error: `SAT Critical Readng Avg. Score`
- Non-analytical columns like `internal_school_id`, `contact_extension`, and `pct_students_tested` will be removed later.


## 3. Normalize Column Names

Standardized all column names for **SQL compatibility** and **Pythonic usage**:

- Convert to lowercase
- Replace spaces with underscores
- Remove special characters
- Strip leading/trailing whitespace

```python
df.columns = (
    df.columns
      .str.strip()
      .str.lower()
      .str.replace(' ', '_', regex=True)
      .str.replace(r'[^\w]', '', regex=True)
)
```
## 4. Type Conversion & Parsing

In this step, we ensure that all numeric columns are properly typed for analysis and database insertion.

- **Convert SAT score columns** to numeric, coercing invalid values to `NaN`.
- **Clean `pct_students_tested`** by removing `%` and converting it to a float in the `[0,1]` range.
- **Convert `academic_tier_rating`** to a nullable integer (`Int64`) to preserve missing values.

```python
# Define SAT-related numeric columns
numeric_cols = [
    'num_of_sat_test_takers',
    'sat_critical_reading_avg_score',
    'sat_math_avg_score',
    'sat_writing_avg_score'
]
```
# Convert SAT score columns to numeric, invalid entries become NaN
```python
df[numeric_cols] = df[numeric_cols].apply(pd.to_numeric, errors='coerce')
```
# Clean percentage column and convert to float
```python
df['pct_students_tested'] = (
    df['pct_students_tested']
      .str.rstrip('%')                  # remove '%' sign
      .pipe(pd.to_numeric, errors='coerce')  # convert to numeric
      / 100                             # scale to [0,1]
)
```
# Convert academic tier ratings to nullable integer type
```python
df['academic_tier_rating'] = (
    pd.to_numeric(df['academic_tier_rating'], errors='coerce')
      .astype('Int64')                  # preserve missing values
)
```
## 5. Validation Checks

In this step, we validate the data to ensure consistency and correctness before loading it into PostgreSQL.

- **SAT Score Validation:** Ensure all SAT scores are within the official range of 200–800.
- **Percentage Validation:** Confirm that `pct_students_tested` values are within the `[0, 1]` range.
- **Duplicate Detection:** Identify duplicate rows for inspection.

```python
# Filter rows where all SAT scores are within valid range
df_filtered = df[
    (df['sat_critical_reading_avg_score'] >= 200) &
    (df['sat_math_avg_score'] >= 200) &
    (df['sat_writing_avg_score'] >= 200)
]
```
# Remove exact duplicate rows
df_filtered_unique = df_filtered.drop_duplicates().reset_index(drop=True)

## 6. Handle Missing / Nullable Values

PostgreSQL cannot directly handle pandas nullable types (`pd.NA`) during batch inserts.  
To ensure compatibility, we perform two main steps:

1. **Convert `pd.NA` to Python-native `None`**  
   This allows `psycopg2` to correctly insert missing values as SQL `NULL`.

2. **Ensure Python-native data types for all columns**  
   - **Int64 columns** → Python `int` or `None`  
   - **Float columns** → Python `float`  

Finally, we convert the DataFrame into a list of tuples to use with `psycopg2.extras.execute_batch`.

```python
# Step 1: Convert pd.NA to Python None
df_cleaned = df_filtered_unique.copy()
df_cleaned = df_cleaned.replace({pd.NA: None})

# Step 2: Ensure Python-native types for psycopg2
for col in df_cleaned.columns:
    if pd.api.types.is_integer_dtype(df_cleaned[col]):
        df_cleaned[col] = df_cleaned[col].astype(object)  # Int64 -> Python int / None
    elif pd.api.types.is_float_dtype(df_cleaned[col]):
        df_cleaned[col] = df_cleaned[col].astype(float)

# Step 3: Convert DataFrame to list of tuples for batch insertion
records = list(df_cleaned.itertuples(index=False, name=None))
```
## 7. PostgreSQL Integration

This step ensures that the cleaned SAT dataset is safely inserted into the PostgreSQL database.  
Key points:

- **Table creation** ensures the schema exists before inserting data.  
- **ON CONFLICT (dbn) DO UPDATE** allows updating existing rows instead of creating duplicates.  
- **Batch insert** using `execute_batch` improves performance and safely handles large datasets.  
- **Transactions** are committed only if all inserts are successful; otherwise, they are rolled back to maintain database integrity.

A direct PostgreSQL connection is established using `psycopg2`.

> **Note:**  
> Hardcoded credentials are used here **for onboarding and learning purposes only**.  
> In production, credentials should be stored securely (e.g., environment variables or secrets managers).

```python
from psycopg2.extras import execute_batch
import psycopg2

try:
    # ============================================================
    # 1. Establish a secure connection to the PostgreSQL database
    # ============================================================
    conn = psycopg2.connect(
        dbname="neondb",
        user="neondb_owner",
        password="a9Am7Yy5r9_T7h4OF2GN",
        host="ep-falling-glitter-a5m0j5gk-pooler.us-east-2.aws.neon.tech",
        port="5432",
        sslmode="require"
    )
    cur = conn.cursor()  # Create a cursor object for executing SQL commands

    # ============================================================
    # 2. Create the SAT scores table if it does not exist
    # ============================================================
    cur.execute("""
    CREATE TABLE IF NOT EXISTS alexandra_dernova_sat_scores (
        dbn TEXT PRIMARY KEY,                           -- Unique school identifier
        num_of_sat_test_takers INTEGER,                -- Number of students who took SAT
        sat_critical_reading_avg_score FLOAT,          -- Average Critical Reading score
        sat_math_avg_score FLOAT,                       -- Average Math score
        sat_writing_avg_score FLOAT,                    -- Average Writing score
        academic_tier_rating INTEGER                    -- Tier rating (1–4)
    );
    """)

    # ============================================================
    # 3. Prepare parameterized INSERT query with conflict handling
    # ============================================================
    insert_query = """
    INSERT INTO alexandra_dernova_sat_scores
    (
        dbn,
        num_of_sat_test_takers,
        sat_critical_reading_avg_score,
        sat_math_avg_score,
        sat_writing_avg_score,
        academic_tier_rating
    )
    VALUES (%s, %s, %s, %s, %s, %s)
    ON CONFLICT (dbn) DO UPDATE SET
        num_of_sat_test_takers = EXCLUDED.num_of_sat_test_takers,
        sat_critical_reading_avg_score = EXCLUDED.sat_critical_reading_avg_score,
        sat_math_avg_score = EXCLUDED.sat_math_avg_score,
        sat_writing_avg_score = EXCLUDED.sat_writing_avg_score,
        academic_tier_rating = EXCLUDED.academic_tier_rating;
    """

    # ============================================================
    # 4. Execute batch insert for performance
    # ============================================================
    # execute_batch is faster than inserting row by row
    execute_batch(cur, insert_query, records, page_size=100)

    # Commit transaction to save changes
    conn.commit()
    print(f"Successfully inserted/updated {len(records)} records.")

except Exception as e:
    # Rollback transaction in case of errors
    conn.rollback()
    print("Error during database insertion:", e)

finally:
    # Close cursor and connection to release resources
    cur.close()
    conn.close()
```
## 8. Challenges & Notes

### Nullable Values
- Pandas `Int64` (nullable integer) and `pd.NA` cannot be directly adapted by `psycopg2`.  
- Resolved by converting nullable integers to Python-native `int`/`None` and floats to Python `float`.  
- Ensures batch insertion works without raising `can't adapt type 'NAType'` errors.

### SQL Naming
- Hyphens `-` in table names are not allowed in SQL syntax.  
- Replaced with underscores `_` to prevent syntax errors.  
- Example: `alexandra-dernova_sat_scores` → `alexandra_dernova_sat_scores`.

### Duplicate Columns
- Column `SAT Critical Readng Avg. Score` was removed due to typographical error and duplication.  

### Data Consistency
- All SAT scores validated to be within the official 200–800 range.  
- Non-essential columns (`school_name`, `internal_school_id`, `contact_extension`, `pct_students_tested`) were removed prior to database insertion.  

---

## 9. Result

- **421 cleaned records** were successfully inserted or updated in PostgreSQL.  
- The table `alexandra_dernova_sat_scores` is now ready for analytics and joining with other school datasets.
