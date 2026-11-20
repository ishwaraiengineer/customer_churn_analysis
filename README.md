# Customer Churn Analysis

[![Python](https://img.shields.io/badge/Python-3.x-blue?logo=python&logoColor=white)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458?logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![NumPy](https://img.shields.io/badge/NumPy-Numerical%20Computing-013243?logo=numpy&logoColor=white)](https://numpy.org/)
[![SQLite](https://img.shields.io/badge/SQLite-Database-003B57?logo=sqlite&logoColor=white)](https://www.sqlite.org/)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-Visualization-11557c)](https://matplotlib.org/)
[![Seaborn](https://img.shields.io/badge/Seaborn-Visualization-4c72b0)](https://seaborn.pydata.org/)

An exploratory customer churn analysis that combines customer, subscription, and support records to assess retention, compare churn across customer segments, and examine the relationship between support escalations and churn.

---

## Table of Contents

- [Executive Summary](#executive-summary)
- [Business Problem](#business-problem)
- [Project Objectives](#project-objectives)
- [Dataset Overview](#dataset-overview)
- [Technology Stack](#technology-stack)
- [Analytical Workflow](#analytical-workflow)
- [Data Cleaning and Preparation](#data-cleaning-and-preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [SQL and Database Work](#sql-and-database-work)
- [Statistical Analysis](#statistical-analysis)
- [Key Business Questions](#key-business-questions)
- [Key Findings and Insights](#key-findings-and-insights)
- [Business Recommendations](#business-recommendations)
- [Project Architecture](#project-architecture)
- [Installation and Setup](#installation-and-setup)
- [How to Run](#how-to-run)
- [Dashboard and Visualizations](#dashboard-and-visualizations)
- [Limitations](#limitations)
- [Future Improvements](#future-improvements)
- [Conclusion](#conclusion)
- [Author](#author)

## Executive Summary

This project examines customer churn using three SQLite tables covering customer attributes, subscription details, and support interactions. Python is used to extract the tables, clean and standardize selected fields, engineer churn-related features, combine the data, calculate business KPIs, and explore patterns through visual analysis.

The resulting analysis dataset contains **21 customer/subscription records**. The observed overall churn rate is **28.57%**, while the retention rate is **71.43%**. Churn differs across subscription plans and acquisition categories in this sample. The analysis also reports a positive correlation between support escalations and churn, which is treated as an exploratory association rather than evidence of causation.

The dataset is small, so the findings are directional and should be validated against a larger customer population before operational decisions are made.

## Business Problem

Customer churn reduces the recurring customer base and can affect subscription revenue. A business needs to understand the scale of churn and whether it is concentrated in particular plans, acquisition channels, or customer groups.

Customer information, subscription records, and support interactions provide different perspectives on the customer relationship. Bringing these sources together makes it possible to explore churn alongside plan attributes and available support indicators.

This analysis addresses the following business need:

> Establish a descriptive view of customer churn, identify segments with higher observed churn, and explore whether support escalations are associated with customer cancellation.

## Project Objectives

- Extract customer, subscription, and support data from a SQLite database.
- Inspect table structures, column types, and missing values.
- Standardize selected customer attributes and convert date fields to datetime.
- Derive a binary churn indicator from cancellation dates.
- Combine the three datasets using `customerid`.
- Calculate churn, retention, plan-level churn, acquisition-type churn, tenure, and revenue-related metrics.
- Examine support complaint and escalation information.
- Explore relationships among churn-related variables using correlation analysis.
- Communicate the results through charts and grouped summaries.

## Dataset Overview

### Data source

The analysis reads from a local SQLite database named `customer_churn.db`. The notebook also contains a commented reference to an Excel file named `customer_churn_data_raw.xlsx`, but the active extraction workflow uses SQLite.

The original dataset publisher, download URL, and licence are not identified in the available project material. These should be added to the repository when verified.

### Source tables

| Table | Rows observed | Columns | Role |
|---|---:|---:|---|
| `db_customer` | 21 | 8 | Customer profile and demographic attributes |
| `db_subscription` | 21 | 11 | Subscription, plan, cancellation, and churn-score information |
| `db_support` | 9 | 6 | Complaint dates, escalation indicators, and CSAT scores |

The customer and subscription tables each contain 21 distinct customer IDs. The support table contains 9 records associated with 7 distinct customers, meaning some customers have multiple support records.

### Important fields

| Field | Description |
|---|---|
| `customerid` | Customer identifier used to join the source tables |
| `name` | Customer name; renamed to `customer_name` during cleaning |
| `country`, `state`, `gender`, `dob` | Customer attributes |
| `subscription_start_date` | Subscription start date |
| `subscription_type` | Acquisition/source category recorded as Organic, Paid, or Refferal |
| `renewal_date` | Subscription renewal date |
| `plan_type` | Basic, Standard, or Premium plan |
| `contract_type` | Contract category, including Monthly and Annual |
| `cancellation_date` | Cancellation date; used to derive the churn flag |
| `cancellation_reason` | Recorded reason for cancellation |
| `monthly_charges` | Monthly subscription charge |
| `cltv` | Customer lifetime value field present in the source |
| `churn_score` | Existing churn score supplied in the source |
| `complaint_date` | Date of a support complaint |
| `escalations` | Support escalation indicator (`Y`/`N`) |
| `csat_score` | Customer satisfaction score |

### Analysis dataset

After preparing the support records and merging the three tables, the notebook produces a DataFrame with **21 rows and 21 columns**. The resulting dataset is also exported as `exported_churn_data.csv`.

### Data considerations

- The source contains only 21 customer/subscription records.
- The support table is one-to-many relative to customers: 9 records represent 7 customers.
- The analysis retains one support row per customer after sorting by complaint date and dropping duplicate customer IDs. Because the sort is ascending by default, this retains the earliest complaint record for each customer with multiple records.
- The dataset source and licence are not documented.
- The notebook labels the revenue-at-risk output as Rs 'k', but the precise unit/scale of `monthly_charges` should be confirmed before the metric is used in a business report.

## Technology Stack

| Category | Technology | Use in this project |
|---|---|---|
| Programming language | Python | Data extraction, cleaning, feature engineering, and analysis |
| Data manipulation | Pandas | DataFrames, joins, grouping, pivot tables, and CSV export |
| Numerical computing | NumPy | Churn flag and risk-category derivation |
| Database | SQLite | Storage and extraction of source tables |
| Database integration | `sqlite3`, `pandas.read_sql()` | Database connections and SQL query execution |
| Visualization | Matplotlib | Trend and categorical bar charts |
| Visualization | Seaborn | Heatmap, pairplot, and categorical faceting |
| Development environment | Jupyter Notebook | Interactive analysis and code execution |

No Streamlit dashboard, Power BI report, or machine-learning model is implemented in the documented workflow.

## Analytical Workflow

1. **Database connection:** Open `customer_churn.db` with Python's `sqlite3` module.
2. **Schema discovery:** Query `sqlite_master` to list tables and use `PRAGMA table_info` to inspect table columns.
3. **Data extraction:** Load each source table into a Pandas DataFrame.
4. **Customer data preparation:** Rename the name field, remove unused columns, convert date of birth, standardize gender labels, and fill missing country values using a state-to-country mapping.
5. **Subscription preparation:** Convert subscription, renewal, and cancellation date fields to datetime.
6. **Support preparation:** Remove unused columns and convert complaint dates to datetime.
7. **Feature engineering:** Derive `churn_flag` from `cancellation_date`; calculate complaint counts; derive tenure and churn-risk categories.
8. **Data integration:** Join subscription and customer data, then attach prepared support information using `customerid`.
9. **KPI analysis:** Calculate overall churn and retention, segment churn rates, average monthly charges, tenure, revenue at risk, escalation rate, and average complaint count.
10. **Exploratory analysis:** Compare churn across plans, states, and subscription types; examine cancellation timing and variable relationships.
11. **Visualization and summary:** Produce charts, correlation heatmaps, and pivot-table summaries.
12. **Export:** Save the combined analysis dataset to `exported_churn_data.csv`.

## Data Cleaning and Preparation

### Customer table

- Renamed `name` to `customer_name`.
- Dropped `interests` and `pincode`; `pincode` contained no non-null values, while `interests` was sparsely populated and not used in the analysis.
- Converted `dob` to datetime.
- Standardized inconsistent gender labels:
  - `Men` and `male` to `Male`
  - `Women`, `female`, and the misspelled `feamale` to `Female`
- Filled missing country values using a mapping derived from states with known country values.

### Subscription table

- Converted `subscription_start_date`, `renewal_date`, and `cancellation_date` to datetime.
- Created `churn_flag` using cancellation-date availability:
  - `1` when `cancellation_date` is present
  - `0` when `cancellation_date` is missing

### Support table

- Dropped `col_1` and `comment` from the analysis.
- Converted `complaint_date` to datetime.
- Calculated complaint counts by `customerid`.
- Sorted support records by complaint date and dropped duplicate customer IDs before the final merge, retaining one support record per customer.

### Data integration

The notebook uses left joins on `customerid`:

- Subscription data joined with customer data.
- The result joined with the deduplicated support data.

An initial merge before support deduplication produced 23 rows because some customers had multiple support records. After deduplicating support to one row per customer and rebuilding the merge, the analysis dataset contained 21 rows and 21 columns.

The final CSV export is created with:

```python
df.to_csv("exported_churn_data.csv", index=False)
```

## Exploratory Data Analysis

The analysis explores churn from several business perspectives.

### Churn by subscription plan

The churn rate is calculated as the mean of `churn_flag` within each plan and expressed as a percentage. This allows plan-level comparisons.

### Churn by state

The notebook groups customers by state and calculates the mean churn flag. This provides a geographic comparison, although the small segment counts make these percentages unstable.

### Churn by subscription type

The analysis compares churn across Organic, Paid, and Refferal subscription categories to explore whether observed retention differs by acquisition/source type.

### Cancellation trend

Cancellation dates are converted to monthly periods and grouped to show the number of churned customers by cancellation month.

### Plan-level pivot analysis

Pivot tables summarize churn rate, unique customer count, and total monthly charges by plan. This provides context for interpreting percentages alongside segment size and charges.

## SQL and Database Work

SQL is used in two distinct ways.

### Churn database inspection

The notebook queries SQLite metadata to identify available tables:

```sql
SELECT name
FROM sqlite_master
WHERE type = 'table';
```

It then inspects table schemas using SQLite's `PRAGMA table_info(table_name)` command and loads the source tables into Pandas with `pd.read_sql()`.

### Separate SQL practice workflow

A separate database named `test_database.sqlite` is created for a small `users` table exercise. The notebook demonstrates:

- Creating a table with `CREATE TABLE`.
- Inserting sample rows with `INSERT INTO`.
- Retrieving records with `SELECT`.
- Aggregating values by country with `SUM()` and `GROUP BY`.
- Reading query results into Pandas.
- Committing transactions and closing the database connection.

Example aggregation:

```sql
SELECT
    country,
    SUM(budget) AS total_budget
FROM users
GROUP BY country;
```

This exercise demonstrates basic SQL execution through Python and SQLite. It is separate from the customer churn analysis: the documented churn KPIs are calculated in Pandas, and no dedicated churn-analysis SQL query set, CTEs, window functions, or SQL joins are shown.

## Statistical Analysis

### Descriptive metrics

The notebook calculates:

- Overall churn and retention rates.
- Churn rate by plan, state, and subscription type.
- Mean monthly charges, labelled as ARPU.
- Average customer tenure in days.
- Sum of monthly charges for customers marked as churned, labelled as revenue at risk.
- Escalation rate and average complaint count.

### Feature engineering for analysis

- `tenure_days` is calculated from subscription start to cancellation for churned customers. For customers without a cancellation date, it is calculated to the date the notebook is executed.
- `churn_risk` is derived from the existing `churn_score`:
  - Low: score below 50
  - Medium: score from 50 to below 70
  - High: score of 70 or above
- The notebook converts selected categorical fields to numeric category codes to support correlation visualization.

### Correlation analysis

A Pearson correlation is calculated between the numeric escalation indicator and `churn_flag`. The notebook also creates a correlation heatmap for selected encoded variables.

The correlation is exploratory. It does not demonstrate that escalations cause churn, and its value is sensitive to the treatment of missing escalation records and the support-record deduplication approach.

No hypothesis tests, confidence intervals, regression model, or predictive model evaluation are documented.

## Key Business Questions

| Business question | Analytical purpose |
|---|---|
| What is the overall customer churn rate? | Establish a baseline measure of customer loss. |
| What proportion of customers are retained? | Quantify the share of customers without a recorded cancellation. |
| Which subscription plan has the highest observed churn? | Identify plans for further product, pricing, or service investigation. |
| Does churn differ by acquisition/source category? | Compare observed retention across Organic, Paid, and Refferal categories. |
| Does churn vary by state? | Explore geographic differences while considering segment size. |
| How many cancellations occur in each month? | Understand the timing of observed cancellations. |
| What is the average monthly charge? | Establish a basic average-charge measure for the analysed records. |
| What monthly charges are associated with churned customers? | Estimate the monthly charges attached to records marked as churned. |
| Are support escalations associated with churn? | Explore whether support escalation deserves deeper investigation. |
| How many complaints are recorded per customer? | Understand the level of support activity in the source records. |

## Key Findings and Insights

The following results are the values reported by the notebook. They describe this 21-record sample and should not be interpreted as stable population estimates.

### Overall customer metrics

| Metric | Observed result |
|---|---:|
| Customer/subscription records in final analysis | 21 |
| Customers with a recorded cancellation | 6 |
| Overall churn rate | **28.57%** |
| Retention rate | **71.43%** |
| Average monthly charges (labelled ARPU) | 18.85 |
| Average tenure | 1,563 days |
| Monthly charges associated with churned customers (labelled revenue at risk) | 73.94 |
| Escalation rate | 19.05% |
| Average complaint count per customer | 0.43 |
| Escalation/churn correlation reported | 0.77 |

### Churn by plan

| Plan | Customers | Churn rate | Total monthly charges |
|---|---:|---:|---:|
| Basic | 5 | **60.00%** | 52.95 |
| Premium | 7 | **14.29%** | 218.93 |
| Standard | 9 | **22.22%** | 123.91 |

**Insight:** Basic has the highest observed churn rate in this sample. The difference is worth investigating, but the plan contains only five customers, so a small number of records materially affects the percentage.

### Churn by subscription type

| Subscription type | Observed churn rate |
|---|---:|
| Organic | 0.00% |
| Paid | 16.67% |
| Refferal | **83.33%** |

**Insight:** The Refferal category has the highest observed churn rate in the sample. Confirm the underlying segment counts and the intended meaning of the source category before making acquisition-channel decisions.

### Churn by state

| State | Observed churn rate |
|---|---:|
| Delhi | 25.00% |
| Karnataka | 100.00% |
| Kathmandu | 0.00% |
| Maharashtra | 0.00% |
| Meghalaya | 66.67% |
| Nagaland | 0.00% |
| Rajasthan | 0.00% |
| Telangana | 50.00% |
| Uttar Pradesh | 0.00% |

**Insight:** The displayed state-level percentages vary considerably. Some groups are very small; a 100% churn rate in a small segment should not be treated as evidence of a persistent geographic issue.

### Support and churn relationship

The notebook reports an escalation/churn correlation of **0.77** and an escalation rate of **19.05%**.

**Insight:** The positive association suggests that support escalation is a useful area for further investigation. However, the support table contains nine records for seven customers, and the final analysis retains only one support record per customer. Missing escalation values are also converted to zero by the notebook's `np.where()` encoding. These choices can affect the reported rate and correlation.

### Cancellation timing

The monthly churn chart shows cancellations in February, May, September, October, and November 2024, with two cancellations in September and one in each of the other displayed months.

**Insight:** The chart describes the timing of six observed cancellations. The sample is too small to support a reliable seasonal or recurring trend conclusion.

### Metric interpretation

- The notebook calculates ARPU as the simple mean of `monthly_charges`. The billing period and currency scale should be confirmed before using it as a finance reporting metric.
- The notebook prints revenue at risk with an `Rs 'k'` label and calculates it as the sum of monthly charges for churned records. It is a sum of associated monthly charges, not a lifetime revenue-loss estimate or forecast.
- Average tenure depends on the notebook execution date for customers without a cancellation date, so it changes when the analysis is rerun.
- The churn-risk category is derived from the existing `churn_score`; it is not a separately trained prediction.
- The correlation matrix includes encoded categorical variables. Numeric category codes impose an ordering and should not automatically be interpreted as meaningful continuous measurements.

## Business Recommendations

The following are evidence-informed investigation priorities, not claims that an intervention has been tested.

1. **Investigate Basic plan churn.** Review cancellation reasons, contract terms, pricing, plan benefits, and usage patterns for Basic customers. Compare the results against a larger cohort before changing the offer.
2. **Validate referral acquisition quality.** Confirm the definition and record counts for the Refferal category. Analyse retention by acquisition cohort and customer tenure before changing channel spend.
3. **Review escalated support cases.** Examine complaint history, escalation outcomes, resolution time, and CSAT together. Preserve all support events rather than retaining only one record per customer.
4. **Improve segment reporting.** Display customer counts next to churn percentages so stakeholders can distinguish meaningful patterns from small-sample variation.
5. **Standardize churn and revenue definitions.** Define the observation period, active-customer denominator, currency, and reporting scale. Separate monthly recurring revenue at risk from lifetime value or forecasted loss.
6. **Investigate cancellation reasons.** Use the available `cancellation_reason` field to categorize the reasons for churn and identify potential retention actions.

## Project Architecture

The exact repository directory structure is not established by the available project material. The following is a suggested GitHub organization for the existing workflow, not a representation of verified files:

```text
customer-churn-analysis/
├── data/
│   ├── raw/
│   │   └── customer_churn.db
│   └── processed/
│       └── exported_churn_data.csv
├── notebooks/
│   └── customer_churn_analysis.ipynb
├── visuals/
│   └── [exported chart images]
├── README.md
└── requirements.txt
```

The analysis also creates a separate `test_database.sqlite` file for the SQL practice exercise. Include it only if you intend to preserve that demonstration in the repository.

Do not commit private customer data, credentials, or database files unless you have the right to publish them. If the database cannot be shared, provide an approved sample or documented data-generation process.

## Installation and Setup

The notebook uses Python with NumPy, Pandas, Matplotlib, Seaborn, and SQLite. SQLite's `sqlite3` module is part of Python's standard library. A dependency lockfile was not part of the supplied project material, so these are setup commands for the libraries observed in the notebook.

### 1. Clone the repository

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd customer-churn-analysis
```

Replace the placeholder with the actual repository URL.

### 2. Create a virtual environment

**Windows**

```bash
python -m venv .venv
.venv\Scripts\activate
```

**macOS / Linux**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
python -m pip install numpy pandas matplotlib seaborn jupyter
```

### 4. Place the database

Place `customer_churn.db` in the location expected by the notebook, or update the SQLite connection path. The notebook uses a relative path:

```python
conn = sqlite3.connect("customer_churn.db")
```

The database file must be available locally for the analysis to run.

## How to Run

1. Activate the virtual environment.
2. Ensure `customer_churn.db` is in the expected working directory.
3. Launch Jupyter:

```bash
jupyter notebook
```

4. Open the customer churn analysis notebook.
5. Run the cells in order, from database extraction through analysis and visualization.

The notebook exports the final merged dataset to `exported_churn_data.csv` in the current working directory.

The actual notebook filename and repository paths should be updated in this section to match the final GitHub repository. No standalone Python entry-point script or Streamlit application is evidenced in the documented workflow.

## Dashboard and Visualizations

This project contains analytical visualizations, but no interactive dashboard application is implemented in the documented workflow.

The visual analysis includes:

- Monthly cancellation trend line chart.
- Churn rate by subscription plan.
- Churn rate by state.
- Correlation heatmap for selected encoded variables.
- Seaborn pairplot for relationships among selected variables.
- Seaborn categorical facet plot comparing plan type, monthly charges, gender, and churn-risk category.
- Pivot-table summaries of churn, customer counts, and monthly charges by plan.

Export the final charts into a repository folder such as `visuals/` and embed them here using verified relative paths. No image paths are included until the image files exist in the repository.

## Limitations

- **Small sample:** The analysis contains 21 customer/subscription records and six recorded cancellations. Segment rates are highly sensitive to individual records.
- **Source provenance:** The original dataset source, licence, and download link are not documented.
- **Support history reduction:** Multiple support events are reduced to one record per customer for the final merge. This loses event-level detail, even though complaint counts are calculated before deduplication.
- **Missing support values:** The escalation indicator is encoded with `np.where(escalations == 'Y', 1, 0)`, which also converts missing values to zero. The escalation rate and correlation therefore need a missing-data-aware re-calculation.
- **Correlation is not causation:** The reported 0.77 correlation does not show that support escalation causes churn.
- **Tenure reproducibility:** Active-customer tenure is calculated relative to the current execution date, so the average changes over time.
- **ARPU and revenue units:** The reporting unit and scale of monthly charges are not fully established.
- **Category encoding:** Correlation on integer-encoded categories can imply an artificial ordering.
- **No predictive validation:** The project does not train or evaluate a churn prediction model.
- **Notebook warnings:** The categorical encoding cells produce Pandas `SettingWithCopyWarning`; explicit copies and `.loc` assignments should be used to make the transformations unambiguous.
- **No churn-specific SQL analysis:** The SQL practice section uses a separate sample `users` table; churn KPIs are calculated in Pandas.

## Future Improvements

1. **Retain support event history:** Keep the support table at complaint level and derive customer-level aggregates such as complaint count, escalation count, latest complaint date, and average CSAT.
2. **Improve missing-value treatment:** Preserve missing escalation status as unknown rather than automatically treating it as no escalation.
3. **Validate joins and data grain:** Add checks for key uniqueness, join cardinality, row counts, and duplicate customer IDs before and after each merge.
4. **Build churn-specific SQL analysis:** Add SQL queries for churn KPIs, segment comparisons, monthly cohorts, and customer-level support summaries. Use CTEs or window functions only where they add value.
5. **Add cohort analysis:** Compare retention by signup month, acquisition channel, plan, and tenure cohort.
6. **Strengthen statistical analysis:** Report segment sample sizes and, when supported by sufficient data, confidence intervals and suitable statistical tests.
7. **Validate financial metrics:** Confirm currency, billing period, and definitions for ARPU, recurring revenue, and revenue at risk.
8. **Develop a Streamlit dashboard:** Once the underlying metrics are validated, add filters and views for overall KPIs, plan comparisons, acquisition channels, cancellation trends, and support indicators.
9. **Add reproducibility assets:** Include a `requirements.txt`, data dictionary, dataset source and licence, and a documented database setup process.
10. **Consider predictive modelling only after data expansion:** If a larger time-stamped dataset becomes available, build a leakage-aware churn model and evaluate it with an appropriate temporal validation strategy.

## Conclusion

This project brings customer, subscription, and support data into a single analytical workflow and uses Python, SQLite, Pandas, NumPy, Matplotlib, and Seaborn to examine customer churn.

The analysis establishes a sample churn rate of 28.57%, highlights higher observed churn in the Basic and Refferal segments, and identifies support escalation as a relationship worth investigating further. It also demonstrates practical data preparation, relational data integration, KPI calculation, exploratory visualization, and basic SQL execution through Python.

The main analytical takeaway is not that the observed segment differences are definitive, but that they identify focused questions for deeper investigation. Larger samples, preserved support history, clearer metric definitions, and stronger validation would be the next steps toward more reliable business decisions.

## Author

**Name:** [Your Name]  
**LinkedIn:** [Your LinkedIn Profile URL]  
**GitHub:** [Your GitHub Profile URL]  
**Email:** [Your Professional Email]
