# NYC Schools Incident Reports Analysis - Google Sheets

This document summarizes the **analysis of incident reports across NYC schools** using a [Google Sheets dataset](https://docs.google.com/spreadsheets/d/1CciDC8jgKkKn7CH-mQXwiT2Ha9YTYIKcsE8GGaj5q1E/edit?usp=sharing)
The goal of the analysis was to **clean, inspect, and summarize school-level incident data**, identify duplicates, calculate aggregate metrics, and highlight potential data quality issues for further investigation.

## Analysis Objectives

The analysis specifically aimed to answer the following questions:

1. **How many total rows are there?**  
   - This provides an overview of the dataset size before cleaning.

2. **How many unique schools are listed? (dbn)**  
   - Identifies the number of distinct schools represented in the data.

3. **What is the most frequent incident type?**  
   - Determines which type of incidents (major crimes, other crimes, non-criminal incidents, property crimes, or violent crimes) occur most often.

4. **What percentage of incidents occurred in the Bronx?**  
   - Measures the borough-level distribution of incidents to highlight areas with higher safety concerns.

---

## 1. Dataset Structure

The dataset contains the following key columns:

- **Identifiers & Metadata**
  - `school_year`, `building_code`, `dbn`, `location_name`, `school_id` (calculated as `LOWER(TRIM(C2)) & "_" & LOWER(TRIM(D2))`)
  - `location_code`, `address`, `borough`, `geographical_district_code`, `register`, `building_name`
- **School & Building Counts**
  - `schools`, `schools_in_building`
- **Incident Counts**
  - `major_n_clean`, `oth_n_clean`, `nocrim_n_clean`, `prop_n_clean`, `vio_n_clean`  
  - `major_n`, `oth_n`, `nocrim_n`, `prop_n`, `vio_n`
- **Aggregate Metrics**
  - `avgofmajor_n_clean`, `avgofoth_n_clean`, `avgofnocrim_n_clean`, `avgofprop_n_clean`, `avgofvio_n_clean`
  - `avgofmajor_n`, `avgofoth_n`, `avgofnocrim_n`, `avgofprop_n`, `avgofvio_n`
  - `total_incidents` = sum of all incident types
  - `total_avg_incidents` = sum of average incidents
- **Geography**
  - `borough_name`, `borough_name_clear` (normalized using conditional logic)
  - `postcode`, `latitude`, `longitude`, `community_board`, `council_district`, `census_tract`, `bin`, `bbl`, `nta`
- **Duplicate Tracking**
  - `duplicates` = COUNTIFS to detect repeated DBN and school year combinations

---

## 2. Data Cleaning Steps

### 2.1 Handling Null or Invalid Values
- Numeric columns (`major_n_clean`, `oth_n_clean`, `nocrim_n_clean`, etc.) were cleaned using `IFERROR(VALUE(...);0)` to replace errors with zero.
- Average calculations (`avgof..._n_clean`) were standardized and converted to numeric types.

### 2.2 Normalizing Borough Names
- Derived `borough_name_clear` using a combination of:
  - Explicit column values
  - Fallback logic using `borough` codes (K, X, Q, M, R)
  - Conditional defaults if information is missing (`UNKNOWN`)

### 2.3 Calculated Columns
- `school_id` = concatenation of trimmed and lowercase `dbn` and `location_name`
- `total_incidents` = sum of all incident types per row
- `total_avg_incidents` = sum of average incidents per row
- `duplicates` = count of rows sharing the same `school_id` (constructed as a normalized combination of `dbn` and `location_name` with trimming and lowercasing) and `school_year`

---

## 3. Summary Statistics

### 3.1 General Counts
- **Total rows:** 6,309  
- **Unique schools (dbn):** 1,890  

### 3.2 Incident Type Frequency
| Incident Type       | Total Occurrences |
|-------------------|-----------------|
| Major crimes       | 1,781           |
| Other crimes       | 6,936           |
| Non-criminal crimes| 11,772          |
| Property crimes    | 4,482           |
| Violent crimes     | 3,180           |

**Most frequent incident type:** Non-criminal crimes

### 3.3 Borough-Level Analysis
- **Percentage of incidents in the Bronx:** 28.3%

| Borough         | Total Incidents | Total Avg Incidents |
|-----------------|----------------|-------------------|
| BRONX           | 7,967          | 6,098.26          |
| BROOKLYN        | 8,522          | 7,826.20          |
| MANHATTAN       | 5,303          | 4,498.37          |
| QUEENS          | 4,439          | 7,814.54          |
| STATEN IS       | 1,920          | 1,725.84          |
| **Total**       | 28,151         | 27,963.21         |

#### **Two notable patterns were observed in the data**
    - First, the total number of incidents varies substantially across boroughs. Brooklyn and the Bronx report the highest total incidents (8,522 and 7,965 respectively), while Staten Island has the lowest (1,920). This suggests that incident occurrence is not evenly distributed across boroughs.
    - Second, the average number of incidents per school shows some anomalies: Queens has a high average (7,801.85) despite a lower total count, and there are a few schools with unknown boroughs reporting very low totals. These observations highlight both differences in reporting and potential data quality issues.
---

## 4. Duplicate Records Analysis

### 4.1 Fully Duplicated Records
- **Example:** DBN `27Q905` (John Adams Evening Hs M/W, 2013–14)
- Two rows identical across all columns
- **Action:** Removed one row to eliminate full duplication

### 4.2 Partial / Conflicting Records
- **Example:** DBN `10X056` (P.S. 056 Norwood Heights, 2015–16)
- Multiple records share DBN, school year, and building but differ in:
  - School name formatting
  - Address/location fields
  - Safety and incident metrics
- **Action:** Records left in dataset but flagged for ambiguity

### 4.3 Interpretation
- Multiple records may indicate:
  - Different reporting sources (school-level vs. building-level)
  - Aggregated vs. school-specific data
  - Partial updates or revisions
- **Next Steps:**
  - Determine authoritative record per school and year
  - Consider merging or aggregating conflicting records based on metadata

---

## 5. Summary & Next Steps

- Dataset contains **6,309 rows** with **1,890 unique schools**  
- Total incidents across all boroughs: **28,151**  
- Total average incidents across all boroughs: **27,963.21**  
- Most frequent incident type: **Non-criminal incidents (11,772 occurrences)**  
- The Bronx accounts for **28.3% of total incidents**  
- **Duplicate handling:** Full duplicates removed, partial duplicates flagged for review


