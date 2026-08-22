# Project QA & Validation

## Overview

The SCM Power BI Dashboard was reviewed at the data, model, DAX, and report-visual levels before finalization.

The objective of the QA process was to confirm that the dashboard calculations, model relationships, visual bindings, and documented business logic were internally consistent.

---

## Dataset Validation

The final dataset contains:

| Metric | Verified Value |
|---|---:|
| Order-line rows | 10,194 |
| Distinct orders | 8,549 |
| Distinct customers | 5,044 |
| Products | 15 |
| Divisions | 3 |
| Regions | 4 |
| Date range | 2021–2024 |

The dataset-level figures were cross-checked against the source sales data.

---

## Data Model Validation

The final Power BI model contains **five active relationships**.

| Relationship | Status |
|---|---|
| FactSales → DimCustomer | Active |
| FactSales → DimProduct | Active |
| FactSales → DimDate | Active |
| DimProduct → DimDivision | Active |
| FactTargets → DimDivision | Active |

No additional relationship was introduced between `FactTargets` and `DimDate`.

This was intentional because the available target dataset represents 2024-only targets.

---

# DAX Validation

The semantic model contains **17 DAX measures**.

They cover:

- Revenue
- Cost
- Gross Profit
- Units Sold
- Orders
- Customers
- Gross Margin
- Average Order Value
- Prior-year Revenue
- YoY Revenue
- Prior-year Gross Profit
- YoY Gross Profit
- Sales Target
- Target Achievement
- Target Variance
- Target Gap

Time-intelligence calculations use the dedicated `DimDate` table.

Example:

```dax
Revenue LY =
CALCULATE (
    [Revenue],
    SAMEPERIODLASTYEAR ( DimDate[Date] )
)
```

---

# Report Visual Validation

The final report contains three pages.

## Page 1 — Supply Chain Performance & Commercial Intelligence

Executive overview containing:

- Revenue
- Gross Profit
- Gross Margin %
- Orders
- Units Sold
- YoY Revenue %
- Revenue and Gross Profit trend
- Division performance
- Regional revenue
- Top products by revenue

### QA Fixes

The headline Gross Profit card was originally bound to:

```text
Gross Profit LY
```

It was corrected to:

```text
Gross Profit
```

The card therefore represents the current-period Gross Profit measure.

The Revenue & Gross Profit trend chart originally displayed data labels across a large number of monthly points.

Data labels were disabled to improve readability while retaining hover tooltips.

---

## Page 2 — Sales & Profitability Intelligence

The page contains:

- Gross Profit
- YoY Revenue %
- Revenue
- Gross Margin %
- YoY Gross Profit %
- Revenue and Gross Margin trend
- Division performance
- Product Revenue vs Gross Profit
- Top products by revenue

The page was reviewed for consistency with the underlying measures.

---

## Page 3 — Product & Regional Intelligence

The page contains:

- Gross Margin %
- Units Sold
- Revenue
- Gross Profit
- Product revenue ranking
- Top products by Gross Profit
- Gross Profit by Region
- Regional performance table

Regional analysis includes:

- Revenue
- Gross Profit
- Gross Margin %

---

# Target Measure Validation

The source target data represents 2024 targets.

Because `FactTargets` has no direct relationship to `DimDate`, the target-dependent calculations explicitly scope revenue to 2024.

### Target Achievement %

```dax
Target Achievement % =
DIVIDE (
    CALCULATE (
        [Revenue],
        DimDate[Year] = 2024
    ),
    [Sales Target],
    0
)
```

### Target Variance

```dax
Target Variance =
CALCULATE (
    [Revenue],
    DimDate[Year] = 2024
) - [Sales Target]
```

### Target Gap %

```dax
Target Gap % =
[Target Achievement %] - 1
```

This prevents a four-year revenue total from being compared against a single-year target.

---

# Headline KPI Validation

The final Page 1 Gross Profit KPI uses:

```text
[Gross Profit]
```

rather than:

```text
[Gross Profit LY]
```

The validated total Gross Profit is:

```text
$93,442.80
```

This is consistent with the Gross Profit measure used elsewhere in the report.

---

# Trend Chart Validation

The Page 1 Revenue & Gross Profit Trend chart was reviewed after formatting changes.

Final state:

- Data labels: **Off**
- Revenue series: Present
- Gross Profit series: Present
- Monthly trend: Present
- Tooltips: Available

This prevents overlapping labels across the 48-month time series.

---

# KPI Consistency

The following headline metrics were checked for consistency:

| KPI | Status |
|---|---|
| Revenue | ✅ |
| Gross Profit | ✅ |
| Gross Margin % | ✅ |
| Orders | ✅ |
| Units Sold | ✅ |
| YoY Revenue % | ✅ |
| YoY Gross Profit % | ✅ |

The core dashboard KPIs remain consistent across the report pages where they are displayed.

---

# Business Result Validation

The final model produces:

| Metric | Value |
|---|---:|
| Revenue | $141,783.63 |
| Gross Profit | $93,442.80 |
| Gross Margin | 65.91% |

Additional verified observations include:

- Chocolate contributes approximately 93% of total revenue.
- Regional gross margins remain within a relatively narrow range.
- YoY Revenue growth and YoY Gross Profit growth are approximately 49.5%.

These figures were checked against the underlying sales data.

---

# Known Limitations

The following issues are documented rather than silently modified:

### 1. Target data is 2024-only

The current target source does not provide a multi-year target history.

### 2. No FactTargets–DimDate relationship

The target calculations therefore explicitly scope revenue to 2024.

### 3. Ship Date anomaly

The source data contains Ship Date values substantially later than Order Date for the supplied records.

This was treated as a characteristic of the source dataset rather than artificially corrected.

### 4. Geographic ZIP dataset not joined

`uszips.csv` is available in the source package but is not currently integrated into the model.

### 5. Some measures are model-only

Not all 17 measures are currently displayed as dashboard visuals.

This is intentional and allows the semantic model to support future report expansion.

---

# Final QA Status

| Area | Status |
|---|---|
| Source data | ✅ Validated |
| Power Query structure | ✅ Documented |
| Data model | ✅ Validated |
| Relationships | ✅ Validated |
| DAX measures | ✅ Documented |
| Page 1 KPI | ✅ Corrected |
| Page 1 trend labels | ✅ Corrected |
| Target calculations | ✅ Corrected |
| Dashboard pages | ✅ Reviewed |
| Business KPIs | ✅ Consistent |
| Known limitations | ✅ Documented |

## Final Status

**Publication-ready portfolio project**, subject to normal GitHub repository and source-data licensing checks.
