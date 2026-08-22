# Power Query / Data Preparation

## Overview

The SCM Power BI Dashboard uses **Power Query (M)** as the data preparation layer between the raw U.S. Candy Distributor CSV files and the final dimensional model.

The transformation approach follows:

```text
Raw CSV Sources
      ↓
Staging Queries
      ↓
Dimension / Fact Queries
      ↓
Power BI Semantic Model
      ↓
DAX Measures
      ↓
Dashboard
```

---

## Staging Layer

The project uses four staging queries:

- `STG_Sales`
- `STG_Products`
- `STG_Targets`
- `STG_Factories`

The staging layer provides a clean source layer for downstream transformations.

The staging queries are intentionally separated from the analytical tables so that transformations can be maintained without directly modifying the raw source structure.

> **Note:** The exact source-import M code for the four staging queries is not reproduced here because it was not directly extractable from the supplied `.pbix` file. The downstream transformation logic reflects the verified project structure and applied transformation workflow.

---

# Dimension Transformations

## DimCustomer

### Source

`STG_Sales`

### Transformation Steps

1. Select customer and geographic attributes.
2. Set appropriate data types.
3. Remove duplicate Customer IDs.
4. Sort by Customer ID.

### Output Columns

- Customer ID
- Country/Region
- State/Province
- City
- Postal Code
- Region

### Purpose

Creates a unique customer dimension for customer and regional filtering.

---

## DimProduct

### Source

`STG_Products`

### Transformation Steps

1. Select product attributes.
2. Set appropriate data types.
3. Remove duplicate Product IDs.
4. Sort by Product ID.

### Output Columns

- Product ID
- Product Name
- Division
- Factory
- Unit Price
- Unit Cost

### Purpose

Provides product-level attributes for product ranking, division analysis, and profitability analysis.

---

## DimDivision

### Source

`STG_Products`

### Transformation Steps

1. Select Division.
2. Set the column to text.
3. Remove duplicate divisions.
4. Sort alphabetically.

### Output Values

- Chocolate
- Other
- Sugar

### Purpose

Provides a dedicated division dimension shared by product and target analysis.

---

## DimDate

### Source

`STG_Sales[Order Date]`

A dedicated calendar table is generated dynamically from the minimum and maximum Order Date.

### Date Range Logic

The calendar starts at the beginning of the year containing the minimum Order Date and ends at the end of the year containing the maximum Order Date.

### Calendar Attributes

The transformation creates:

- Date
- Year
- Quarter
- Quarter Number
- Month
- Month Number
- Month Short
- Year Month
- Year Month Sort
- Month Year
- Week Number
- Day
- Day of Week
- Day of Week Number
- Is Weekend

### Purpose

Provides a continuous date dimension required for:

- Year filtering
- Quarter filtering
- Monthly trend analysis
- Year-over-year calculations
- DAX time intelligence

---

# Fact Transformations

## FactSales

### Source

`STG_Sales`

### Transformation Steps

1. Select transactional fields.
2. Apply appropriate data types.
3. Sort by Order Date and Order ID.

### Output Columns

- Row ID
- Order ID
- Order Date
- Ship Date
- Ship Mode
- Customer ID
- Product ID
- Units
- Sales
- Gross Profit
- Cost

### Grain

The table remains at **order-line level**.

This grain is important because one Order ID can contain multiple order lines.

---

## FactTargets

### Source

`STG_Targets`

### Transformation Steps

1. Select Division and Target.
2. Set data types.
3. Remove duplicate divisions.

### Output Columns

- Division
- Target

### Target Scope

The source target data represents **2024 sales targets**.

The final model does not create a direct relationship between FactTargets and DimDate.

Target measures therefore explicitly scope revenue to 2024 in DAX.

---

# Data Preparation Principles

The Power Query layer follows several modeling principles.

### 1. Separate staging from analytical tables

Raw source queries are kept separate from the dimensional model.

### 2. Apply data types before analytical use

Dates, numeric values, identifiers, and text fields are explicitly typed before being used by downstream tables.

### 3. Deduplicate dimensions

Dimension tables are reduced to unique business entities such as:

- Customer ID
- Product ID
- Division

### 4. Preserve fact-table grain

`FactSales` remains at order-line grain rather than being prematurely aggregated.

This allows DAX measures such as:

```dax
Orders =
DISTINCTCOUNT ( FactSales[Order ID] )
```

to calculate distinct orders correctly.

### 5. Generate the calendar dynamically

`DimDate` derives its boundaries from the sales data rather than relying on manually entered dates.

---

# Transformation Flow

```text
Candy_Sales.csv
      │
      ▼
  STG_Sales
      │
      ├──────────────► DimCustomer
      │
      ├──────────────► DimDate
      │
      └──────────────► FactSales


Candy_Products.csv
      │
      ▼
 STG_Products
      │
      ├──────────────► DimProduct
      │
      └──────────────► DimDivision


Candy_Targets.csv
      │
      ▼
 STG_Targets
      │
      └──────────────► FactTargets


Candy_Factories.csv
      │
      ▼
 STG_Factories
```

---

# Power Query → Model → DAX

The overall analytical pipeline is:

```text
Raw CSV Data
     ↓
Power Query / M
     ↓
Clean Staging Tables
     ↓
Fact & Dimension Tables
     ↓
Power BI Data Model
     ↓
17 DAX Measures
     ↓
Interactive Dashboard
```

Power Query is responsible primarily for **data preparation and modeling**, while DAX is responsible for **business calculations and analytical logic**.

---

# Known Data Limitations

The source dataset contains several characteristics that are intentionally documented rather than silently modified:

- `Candy_Targets.csv` represents 2024-only targets.
- `FactTargets` has no direct relationship with `DimDate`.
- `uszips.csv` is currently not joined to the model.
- Ship Date values contain a significant gap relative to Order Date in the supplied dataset.

These issues are documented as limitations rather than being artificially corrected without source-level evidence.
