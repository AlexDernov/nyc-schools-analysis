# High School Directory Analysis

**Project Area:** Data analysis of New York City high schools  
**Objectives:** Load, clean, filter, aggregate, and visualize high school data, with a focus on Brooklyn and comparative analyses across boroughs.  

---

## 1. Overview

This project analyzes the dataset `high-school-directory.csv`, which contains comprehensive information about high schools in New York City. The focus is on the following tasks:

- Data cleaning and standardization of column names  
- Selection of relevant variables for analysis  
- Filtering by location (boroughs) and grade levels  
- Aggregation and summarization of schools per borough  
- Creation of visualizations to support the analysis  

---

## 2. Data Loading

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv("day_2_datasets/high-school-directory.csv")
df.head()
```

**Result:**
Column names are now consistent (e.g., `school_name`, `grade_span_min`, `total_students`) and are easily usable for analysis and aggregation.

---
## 3Selection of Relevant Columns and Data Quality

**Relevant columns for this analysis:**

 - borough → for geographic grouping

 - dbn → unique school identifier

 - grade_span_min / grade_span_max → grade levels

 - total_students → student count for aggregation

**Missing Values Check:**
```text
borough           0
dbn               0
grade_span_min    3
grade_span_max    0
total_students    9
```
## 4. Data Quality Notes

 - borough and dbn contain no missing values → safe to use

 - grade_span_min has few missing values → rows are dropped only for grade-related analyses

 - total_students has 9 missing values → Pandas automatically ignores NaN when computing averages

**Validity Check:**

 - No duplicate dbn

 - No invalid grade ranges (min > max)

## 5. Filtering by Borough

**Select Brooklyn schools:**
```python
df_brooklyn = df[df["borough"].str.lower() == "brooklyn"]
```

**Result:**
121 unique schools in Brooklyn

## 6. Schools Accepting 9th Grade

**Grade Information:**
```text
grade_span_min
9.0    351
6.0     79
7.0      2

grade_span_max
12    404
11     19
10      9
9       3
```

**Filtering:** Only schools that accept 9th grade, after dropping rows with missing grade values
```python
schools_accepting_9th = brooklyn_grades[
    (brooklyn_grades["grade_span_min"] <= 9) &
    (brooklyn_grades["grade_span_max"] >= 9)
]
```

**Result:** All 121 Brooklyn schools accept 9th grade

## 7. Grouping and Aggregation by Borough
### 7.1 Number of Schools per Borough
```python
schools_per_borough = df.groupby('borough')['dbn'].nunique().sort_values(ascending=False)
```
    - Brooklyn: 121
    - Bronx: 118
    - Manhattan: 106
    - Queens: 80
    - Staten Island: 10

### 7.2 Average Students per School
```python
avg_students_per_borough = df.groupby("borough")["total_students"].mean().round(1)
```
    - Bronx: 490
    - Brooklyn: 699
    - Manhattan: 590
    - Queens: 1,047
    - Staten Island: 1,848

### 7.3 Summary of Maximum Grade
```python
grade_span_summary = df.groupby("borough")["grade_span_max"].describe()
```

**Result:** Almost all schools reach grade 12, with minimal differences between boroughs.

### 7.4 Consolidated Summary per Borough
```python
summary_df = pd.DataFrame({
    "Number of Schools": df.groupby("borough")["dbn"].nunique(),
    "Average Students": df.groupby("borough")["total_students"].mean().round(1),
    "Grade Max Mean": df.groupby("borough")["grade_span_max"].mean().round(2),
    "Grade Min Mean": df.groupby("borough")["grade_span_min"].mean().round(2)
}).sort_values("Number of Schools", ascending=False)
```
| Borough       | Number of Schools | Average Students | Grade Max Mean | Grade Min Mean |
|---------------|-----------------|----------------|----------------|----------------|
| Brooklyn      | 121             | 699.1          | 11.93          | 8.43           |
| Bronx         | 118             | 490.4          | 11.91          | 8.42           |
| Manhattan     | 106             | 589.8          | 11.88          | 8.48           |
| Queens        | 80              | 1,046.6        | 11.82          | 8.39           |
| Staten Island | 10              | 1,847.5        | 12.00          | 9.00           |

## 8. Key Insights

  1. Brooklyn has the most high schools (121), followed by Bronx and Manhattan.

  2. All Brooklyn schools accept 9th grade, showing broad coverage from the typical high school entry point.

  3. Average starting grades vary:
    - Brooklyn: 8.43 → some schools integrate middle school (grades 6–8)
    - Queens: 8.39
    - Staten Island: 9.00 → pure high schools

  4. Student count per school varies significantly across boroughs:
    - Staten Island has few, but large schools
    - Brooklyn and Manhattan have many, moderately sized schools

  5. Grade structures are consistent, almost all schools reach grade 12.

## 9. Visualizations

- Number of Schools per Borough 
  ![Bar chart showing the number of high schools per borough](visualizations/bar_number_of_schools_per_borough.png)
- Average Students per School
  ![Bar chart showing the average number of students per schools by borough](visualizations/bar_average_number_of_students_per_school_by_borough.png)


## 10. Conclusion

The analysis shows a clear differentiation between boroughs in terms of the number of schools, school size, and grade coverage. **Brooklyn** stands out with **the highest number of schools**, whereas **Staten Island** has **few but very large schools**. The data suggest that **some schools integrate combined middle and high school programs**, which is relevant for educational planning.