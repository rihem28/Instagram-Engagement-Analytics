# Instagram Engagement Analytics — Outlier & Engagement-Driver Analysis

## 1. Project Overview

**Instagram Engagement Analytics** is an end-to-end Business Intelligence and exploratory analytics project that transforms simulated Instagram posts data into decision-oriented insights through data cleaning, dimensional modeling, statistical outlier detection, and interactive Power BI dashboards.

The project extends an initial Instagram BI analysis by investigating whether apparent differences in engagement rate across content categories, media types, and traffic sources remain consistent after excluding IQR-flagged outliers.

The workflow integrates:

- Python-based data cleaning and KPI engineering
- MySQL star-schema data warehousing
- IQR-based outlier detection
- Power BI dashboards and DAX measures
- Comparative analysis of engagement-rate patterns before and after outlier exclusion

> **Analytical focus:** Assessing the robustness of category-level engagement-rate comparisons and examining the relationship between impressions and engagement rate.

---

## 2. Business Questions

This project investigates the following questions:

- How does engagement vary across content categories, media types, and traffic sources?
- How many posts are flagged as engagement-rate outliers using the IQR method?
- Do category-level engagement-rate comparisons remain consistent after excluding flagged posts?
- How are engagement rate and impressions related?
- Can raw engagement-rate averages be used reliably for content-performance comparisons and recommendations?
  
---

## 3. Dataset

- **Source:** Kaggle — Instagram Analytics Dataset
- **Records:** 30,000 simulated Instagram posts
- **Time period:** December 2024 – November 2025 (12 months)
- **Data level:** Post-level performance metrics
- **Purpose:** Educational and analytical experimentation

Relevant attributes include engagement metrics, impressions, reach, content category, media type, traffic source, upload date, and follower growth.

---

## 4. Technical Stack

| Layer | Tools |
|---|---|
| Data processing | Python, pandas |
| Data staging | CSV |
| Database | MySQL |
| Database connection | mysql-connector-python |
| Business Intelligence | Power BI Desktop |
| Calculations | DAX |
| Outlier detection | Python, IQR method |
| Configuration | python-dotenv |

---

## 5. Data Pipeline & Architecture

The project follows an Extract → Transform → Load (ETL) workflow:

```text
Raw CSV
   ↓
Extraction & Snapshot
   ↓
Data Cleaning & KPI Engineering
   ↓
Dimensional Modeling
   ↓
Processed CSV Files
   ↓
MySQL Star Schema
   ↓
Power BI Dashboards
```

### 5.1 Extract — `Extract.py`

- Loads the raw Instagram dataset from the staging directory.
- Creates a raw snapshot to support traceability before transformation.

### 5.2 Transform — `transform.py`

The transformation process includes:

- Handling missing values using median and mode imputation.
- Validating and converting upload dates.
- Standardizing categorical text values.
- Removing duplicate posts based on `post_id`.
- Engineering engagement and growth-related KPIs.
- Detecting engagement-rate outliers using the IQR method.
- Preparing dimension and fact tables for export.

### 5.3 Load — `Load.py`

- Connects to MySQL using environment-based configuration.
- Creates the dimension and fact tables programmatically.
- Loads processed CSV files into the warehouse.
- Uses `INSERT IGNORE` to support repeated loading.

---

## 6. Data Warehouse Design

The processed data is organized into a MySQL star schema.

```
Time_Dim ──┐
Content_Dim ─┼──< Instagram_Fact >── Media_Dim
Traffic_Dim ─┘
```

### Dimension Tables

- **Time_Dim:** Upload date and time-related attributes.
- **Content_Dim:** Content category, caption length, and hashtag count.
- **Media_Dim:** Media type.
- **Traffic_Dim:** Traffic source.

### Fact Table

**Instagram_Fact** contains post-level metrics, including:

- Likes, comments, shares, saves, and total engagement.
- Reach and impressions.
- Follower growth.
- Engagement rate and growth rate.
- High-engagement flag.
- Average engagement by media type.
- IQR-based outlier flag.

The `is_outlier` field is persisted in the warehouse so Power BI can compare results with and without flagged posts.

---

## 7. Analytical Methodology

### 7.1 IQR-Based Outlier Detection

Outliers were identified using the 1.5 × IQR rule applied to `engagement_rate`.

```python
Q1 = df["engagement_rate"].quantile(0.25)
Q3 = df["engagement_rate"].quantile(0.75)

IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

df["is_outlier"] = (
    ~df["engagement_rate"].between(lower_bound, upper_bound)
).astype(int)
```

**Result:** 2,557 posts were flagged, representing approximately 9% of the dataset.

### 7.2 Categorical Association Screening

Content categories and traffic sources were one-hot encoded before calculating their correlations with engagement rate.

```python
dummies = pd.get_dummies(df[["content_category", "traffic_source"]])
corr = dummies.corrwith(df["engagement_rate"]).sort_values(key=abs, ascending=False)
```

The screening produced absolute correlation values below 0.02 for the examined category and traffic-source indicators.

This indicates negligible measured linear associations in this dataset. However, correlation screening does not establish causality or prove that these dimensions have no influence on engagement.

### 7.3 Comparative Analysis

Power BI was used to compare engagement-rate results:

- Before outlier exclusion.
- After excluding IQR-flagged posts.
- Across content categories, media types, and traffic sources.
- At category-level and post-level granularity.

Log-transformed engagement-rate analysis was also used to examine the relationship between impressions and engagement rate.

### 7.4 Power BI DAX measures

-Avg Engagement (All)
-Avg Engagement (No Outliers)
-Outlier Post Count
-% Outlier Posts
-log avg engagement rate

---

## 8. Power BI Dashboards

The project includes exploratory and analytical dashboard pages covering:

### Overview

- Engagement-rate and engagement KPIs.
- Reach and impressions.
- Engagement by media type.
- Top posts.
- Engagement trends over time.

### Category & Channel Breakdown

- Category-level engagement comparisons.
- Engagement growth trends.
- Reach vs. impressions.
- Traffic-source contribution.

### Performance Details

-Drill-through analysis
-Full comparison analytical table

### Engagement Integrity Check

- Average engagement rate before and after outlier exclusion.
- Category-level comparisons.
- Impressions versus engagement-rate analysis.
- Outlier and non-outlier distributions.

### Further Comparison

- Comparisons by media type and traffic source.
- Outlier percentages across dimensions.
- Supporting comparison tables.
- Drill-through analysis.

Dashboard screenshots are included below to illustrate the main analytical views.

---

## 9. Key Findings

### 1. Engagement-Rate Outliers

The IQR method flagged **2,557 posts, approximately 9% of the dataset**, as engagement-rate outliers.

### 2. Category-Level Ranking Changes

Excluding flagged posts changed category-level engagement-rate comparisons:

- Beauty moved from 1st place to 6th place.
- Travel moved from the lowest raw average to 3rd place.
- The cross-category spread decreased from approximately 13% to 6%.

These changes indicate that category rankings are sensitive to the treatment of engagement-rate outliers.

### 3. Impressions and Engagement Rate

The post-level log-scale analysis showed a continuous impressions–engagement-rate relationship rather than clearly separated populations.

The findings suggest that the IQR classification is strongly associated with the distribution of impressions and the ratio-based nature of engagement rate.

### 4. Interpretation Caution

Engagement-rate averages should not be interpreted independently of reach or impressions. A high engagement rate does not necessarily indicate stronger overall content performance, particularly when the underlying impression count is low.

---

## 10. Business Recommendations

Based on the analysis:

1. **Avoid ranking content categories using engagement rate alone.** Pair engagement rate with impressions, reach, and total engagement.
2. **Review low-impression, high-engagement-rate posts separately** before classifying them as high-performing content.
3. **Use consistent reporting criteria** when comparing categories, media types, and traffic sources.
4. **Recalculate outlier thresholds after data refreshes**, since IQR boundaries depend on the underlying distribution.
5. **Validate the findings on real Instagram data** before using them to guide actual content-strategy decisions.

---

## 11. Limitations

- The dataset is simulated and may not represent real Instagram user behavior.
- Correlation screening does not establish causal relationships.
- IQR identifies statistical outliers but does not determine whether a post is genuinely successful or erroneous.
- Engagement rate is a ratio and should be interpreted alongside its underlying metrics, particularly impressions.
- Findings may change with a different dataset, time period, outlier method, or reporting definition.
- The analysis does not establish that content category, media type, or traffic source has no causal effect on engagement.

---

## 12. Project Structure

```text
Instagram_Project/
│
├── data/
│   ├── staging/
│   └── processed/
│
├── etl/
│   ├── Extract.py
│   ├── transform.py
│   ├── Load.py
│    └── .env
│
├── model/
│    └── star_schema
│
├── Dashboard/
│   └── instagram__BI.pbix
│
├── report/
│   └── Instagram_Analytics_Report.pdf
│
├── presentation
│   └── Instagram_Analytics
│   
├── .gitignore
├── LICENSE
└── README.md
```
---

## 13. How to Run

### 1. Install Dependencies

```bash
pip install pandas mysql-connector-python python-dotenv
```

### 2. Configure Environment Variables

Create a `.env` file containing the required MySQL connection settings:

```env
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_password
DB_NAME=instagram_db
```

### 3. Run the ETL Pipeline

Execute the scripts in the following order:

```bash
python etl/Extract.py
python etl/transform.py
python etl/Load.py
```

### 4. Open the Power BI Report

Open the Power BI file and refresh the data connection to explore the dashboards.

---

## 14. Detailed Report

The accompanying report provides a more detailed explanation of:

- Data preparation and transformation logic.
- Outlier detection methodology.
- DAX calculations.
- Dashboard-level chart interpretations.
- Before-and-after comparison tables.
- Impressions and engagement-rate analysis.
- Analytical limitations and recommendations.

---

## 15. Author

Rihem Abdelmoumen

---
