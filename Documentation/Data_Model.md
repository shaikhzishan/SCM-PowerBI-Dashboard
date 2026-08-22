# Data Model

## Overview

The SCM Power BI Dashboard uses a predominantly **star-schema data model** with a small snowflake structure through `DimDivision`.

The model separates transactional sales data from descriptive dimensions and target data, allowing the report to support revenue, profitability, product, regional, and time-based analysis.

---

## Model Structure

### Fact Tables

#### FactSales

The primary transactional fact table.

**Grain:** one row per order line.

Key fields include:

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

This table provides the foundation for the majority of the dashboard KPIs.

#### FactTargets

Contains division-level sales targets.

Key fields:

- Division
- Target

The available target data represents **2024 targets**.

---

## Dimension Tables

### DimDate

Calendar dimension generated in Power Query from the minimum and maximum Order Date.

The table contains calendar attributes including:

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

`DimDate[Date]` is used for time-intelligence calculations such as:

- Revenue LY
- YoY Revenue
- YoY Revenue %
- Gross Profit LY
- YoY Gross Profit %

### DimCustomer

Customer-level descriptive dimension containing:

- Customer ID
- Country/Region
- State/Province
- City
- Postal Code
- Region

Customer records are deduplicated by Customer ID.

### DimProduct

Product-level dimension containing:

- Product ID
- Product Name
- Division
- Factory
- Unit Price
- Unit Cost

Product records are deduplicated by Product ID.

### DimDivision

Distinct division dimension containing:

- Chocolate
- Other
- Sugar

---

## Relationships

The final model contains **five active relationships**.

| From | To | Relationship |
|---|---|---|
| FactSales[Customer ID] | DimCustomer[Customer ID] | Many-to-one |
| FactSales[Product ID] | DimProduct[Product ID] | Many-to-one |
| FactSales[Order Date] | DimDate[Date] | Many-to-one |
| DimProduct[Division] | DimDivision[Division] | Many-to-one |
| FactTargets[Division] | DimDivision[Division] | Many-to-one |

Relationships use single-direction filtering.

---

## Model Architecture

Conceptually, the model follows this structure:

```text
                    DimDate
                       │
                       │
                       ▼
DimCustomer ──────► FactSales ◄────── DimProduct
                                      │
                                      ▼
                                 DimDivision
                                      ▲
                                      │
                                 FactTargets
