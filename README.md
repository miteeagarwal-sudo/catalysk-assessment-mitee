# catalysk-assessment-mitee

# Catalysk Take-Home Assessment

This repository contains my solution for the Catalysk internship take-home assessment. The project explores a dataset of financial transactions, performs data-quality validation, and analyzes consumer behaviour, outliers, and merchant-level transaction patterns.

## Project Structure

```text
├── README.md
├── requirements.txt
├── transactions.csv
├── explore.py
├── exploration_results.xlsx
├── validate.py
├── validation_report.json
└── analysis.py
```

## Setup

Clone the repository and install the required Python packages:

```bash
pip install -r requirements.txt
```

The project uses Python with:

* pandas
* openpyxl

## How to Run

Run the exploratory analysis:

```bash
python explore.py
```

This prints the main EDA results and saves detailed outputs to:

```text
exploration_results.xlsx
```

Run the validation checks:

```bash
python validate.py
```

This generates:

```text
validation_report.json
```

Run the analytical tasks:

```bash
python analysis.py
```

## Task 1: Exploratory Data Analysis

The raw dataset contains 2,060 rows and 53 distinct consumer identifiers.

The raw transaction date range runs from 2024-01-01 to 2099-12-31, immediately indicating the presence of future-dated records.

Transaction modes are primarily ATM, CARD, IMPS, NEFT, RTGS and UPI, but the field also contains placeholder values such as `N/A`, `null`, `MISSING` and `-`.

The amount column contains 19 non-numeric values represented as `INVALID`.

Among valid DR and CR transactions, credit transactions account for the large majority of total absolute transaction volume, while debit transactions account for a smaller share.

The top five consumers by total debit volume were:

1. USER_10048
2. USER_10015
3. USER_10014
4. USER_10041
5. USER_10009

## Task 2: Data Quality Findings

All seven validation checks identified at least one issue.

| Check | Issue                         | Affected Rows |
| ----- | ----------------------------- | ------------: |
| V-01  | Duplicate transaction IDs     |           116 |
| V-02  | Unparseable transaction dates |            21 |
| V-03  | Non-numeric amounts           |            19 |
| V-04  | DR/CR sign inconsistencies    |            68 |
| V-05  | Future-dated transactions     |            20 |
| V-06  | Empty narrations              |           103 |
| V-07  | Null-like placeholder values  |           114 |

A key implementation choice was to validate the raw dataset before transforming it. This prevents helper columns or derived missing values from being incorrectly counted as source-data quality issues.

## Task 3A: Consumer Behaviour Segmentation

Consumers were segmented using two behavioural dimensions:

* transaction frequency
* average transaction value

Median values were used as data-driven thresholds to produce a 2×2 segmentation:

* High Frequency - High Amount
* High Frequency - Low Amount
* Low Frequency - High Amount
* Low Frequency - Low Amount

Consumers with unusually sparse or extreme behaviour were separated into a `Sparse / Outlier Profile` using the 1.5×IQR rule.

After cleaning, 50 consumers were included in the analysis. The resulting segment distribution was:

| Segment                      | Consumers |
| ---------------------------- | --------: |
| High Frequency - High Amount |        14 |
| High Frequency - Low Amount  |        12 |
| Low Frequency - High Amount  |        10 |
| Low Frequency - Low Amount   |        13 |
| Sparse / Outlier Profile     |         1 |

This segmentation is intentionally interpretable. In a credit-scoring context, these groups represent different behavioural profiles that could be assessed differently without assuming that any segment is automatically low-risk or high-risk.

## Task 3B: Outlier Detection

Outliers were detected at the consumer level using the 1.5×IQR rule across:

* transaction frequency
* total debit volume
* total credit volume
* average transaction value

Invalid consumer identifiers were removed before outlier detection so that data-quality problems were not mistaken for behavioural anomalies.

One valid behavioural outlier was identified: `USER_10000`.

This consumer had 36 transactions, total debit volume of approximately ₹51,521, total credit volume of approximately ₹750,661, and an average transaction value of approximately ₹22,283.

The consumer was flagged because both total credit volume and average transaction value fell outside the IQR-based upper bounds. This would warrant further review, but would not by itself imply fraud or elevated credit risk.

## Task 3C: Merchant-Level Monthly Activity

I examined monthly transaction frequency for merchants/categories with at least 20 valid transactions during 2024.

Nine showed a peak month with transaction activity at least twice their average monthly activity:

* RELIANCE_TRENDS
* NETFLIX
* FLIPKART
* INDIAN_OIL
* MYNTRA
* DMART
* PORTRONICS
* MCD
* BLR_METRO

The strongest monthly concentration was observed for `RELIANCE_TRENDS`, which recorded 9 transactions in July compared with an average of approximately 3.58 transactions per month, giving a peak-to-average ratio of approximately 2.51×.

This suggests that merchant-level spending can show sharp short-term concentrations even when overall consumer behaviour appears relatively stable. A product team could potentially use such merchant-relative spikes for budgeting alerts, unusual-spending notifications, or targeted merchant offers.

Because the dataset covers only one calendar year, I treat these patterns as seasonal-looking monthly concentration rather than evidence of recurring seasonality.

## Assumptions

For debit-versus-credit transaction volume, I use the absolute value of transaction amounts. This prevents negative debit values from cancelling positive credit values and makes the comparison interpretable as total transaction volume.

For analytical tasks, rows with duplicate IDs, malformed consumer IDs, invalid dates, invalid amounts, future dates, invalid transaction types, or DR/CR sign inconsistencies were removed before behavioural analysis.

Merchant/category names were extracted from the middle component of `full_narration`, which generally follows a pattern similar to:

```text
MODE/MERCHANT/REFERENCE
```

## If I Had Another Hour

With additional time, I would make the cleaning pipeline more modular and add automated unit tests for each validation rule. I would also examine merchant patterns over multiple years before making stronger claims about seasonality, and explore whether consumer segmentation remains stable under alternative thresholds or clustering methods. Finally, I would add a small set of visualisations for transaction distributions, segment profiles, and merchant-level monthly activity to make the findings easier for non-technical stakeholders to interpret.
