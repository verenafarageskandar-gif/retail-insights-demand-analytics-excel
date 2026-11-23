# Retail Insights & Demand Analytics (Excel · Power Query · Power Pivot)

An end-to-end Excel analytics project: raw monthly order files and messy reference tables are cleaned with **Power Query**, loaded into a **Power Pivot** data model with a Calendar table, and presented in an interactive dashboard built from PivotTables, PivotCharts and slicers.

> **Dataset note:** This is a training dataset from a data analysis course (2024, 5 Egyptian regions, 5 product categories). It is not real company data.

---

## Table of Contents

1. [Business Questions](#business-questions)
2. [Dashboard Overview](#dashboard-overview)
3. [Dataset](#dataset)
4. [Tools & Skills](#tools--skills)
5. [Data Cleaning (Power Query)](#data-cleaning-power-query)
6. [Data Model (Power Pivot)](#data-model-power-pivot)
7. [Key Figures](#key-figures)
8. [Key Findings](#key-findings)
9. [Repository Structure](#repository-structure)
10. [How to Use](#how-to-use)
11. [Limitations & Possible Improvements](#limitations--possible-improvements)
12. [Author](#author)

---

## Business Questions

- Are we selling more units, or just selling at higher prices? (monthly sales vs. quantity)
- Which products and categories sell consistently throughout the year?
- Which categories perform best in each region?
- How much does each region contribute to total sales?

## Dashboard Overview

**Sheets in the workbook:** `Pivot tables`, `Dashboard`, `orders`, `Products`, `Customers`

**KPI cards:** Total Customers · Total Products Sold · Average Sales per Order · Total Sales

**Visuals:**

| Visual | Purpose |
|---|---|
| Monthly Sales vs. Quantity (by product) | Separates volume effect from price effect |
| Sales Contribution by Region (donut) | Regional share of total sales |
| Total Sales by Category (pie) | Category share of revenue |
| Monthly Quantity by Category (3D bar) | Consistency of demand across months |
| Best-Selling Categories per Region (line) | Category performance within each region |

**Interactivity (slicers):** Region · Category · First Name · Date timeline (months)

Screenshots are available in [`/images`](images/).
<img width="932" height="367" alt="Screenshot 2026-11-23 200944" src="https://github.com/user-attachments/assets/e816fa55-a210-4ad4-a6eb-7af0d5332c8e" />
## Dataset

| Table | Source | Grain | Main fields |
|---|---|---|---|
| `orders` | Folder of monthly files (combined) | One row per order | Order ID, CustomerID, ProductID, Quantity, New Date |
| `Products` | `Products.csv` | One row per product (25 products) | ProductID, ProductName, Category, Price |
| `Customers` | `Customers.csv` | One row per customer (300 customers) | CustomerID, First Name, Last Name, Region |
| `Calendar` | Generated in Power Pivot | One row per date | Date, Year, Month Number, Month, MMM-YYYY, Day Of Week Number, Day Of Week |

- **Regions:** Alexandria, Aswan, Cairo, Giza, Mansoura
- **Categories:** Books, Clothing, Electronics, Furniture, Toys
- **Period:** January – December 2024

## Tools & Skills

- Microsoft Excel (Power Query, Power Pivot, PivotTables, PivotCharts, Slicers, Timeline)
- Data cleaning and transformation (M language)
- Star-schema data modelling
- Dashboard design and data storytelling

## Data Cleaning (Power Query)
<img width="1920" height="1080" alt="Screenshot 2026-11-23 200703" src="https://github.com/user-attachments/assets/7e7e94c0-3c49-4db0-87fd-99e948cba721" />

### `orders` (folder query: combine monthly files)

1. Combined all files from the source folder using the **Transform File** helper function; hidden files filtered out.
2. Extracted the **month number from the file name** (e.g. `..._03.csv` → 3).
3. Set data types (IDs and quantity as whole numbers, date as text for parsing).
4. **Removed duplicates** on `OrderID`.
5. **Removed rows** with a missing `ProductID` or `Quantity`.
6. Standardised date separators (`/` → `-`) and **split the date into three parts**.
7. Rebuilt a clean date, handling mixed formats (day-first and year-first):
   - **Year:** the part that is ≥ 2000
   - **Day:** the part ≤ 31 that does not equal the file's month
   - **Month:** taken from the file name
8. Merged Year-Month-Day into `New Date` (type: Date).
9. Dropped the original date parts and the raw `OrderID`, then generated a new sequential **Order ID** (index).

### `Products`

1. Promoted headers, set types, removed duplicate `ProductID`s.
2. Trimmed whitespace and applied **Proper Case** to `ProductName` and `Category`.
3. Fixed price text issues: `"Two Hundred"` → `200`, removed `$` symbols.
4. Converted `Price` to Currency and rounded up to whole numbers.

### `Customers`

1. Promoted headers, set types, removed duplicate `CustomerID`s.
2. Trimmed whitespace and applied Proper Case.
3. **Split `CustomerName`** into `First Name` and `Last Name`.
4. Cleaned `Region` (removed spaces, trimmed, Proper Case).

The M scripts for each query are stored in [`/power-query`](power-query/).

## Data Model (Power Pivot)
<img width="1920" height="1080" alt="Screenshot 2026-11-22 135422" src="https://github.com/user-attachments/assets/fb859be6-836e-4e45-a71a-1d945f260a65" />

Star schema with `orders` as the fact table:

```
Customers (1) ──────< orders >────── (1) Calendar
                       │
                       └────── (1) Products
```

| Relationship | Key | Cardinality |
|---|---|---|
| Customers → orders | `CustomerID` | One-to-many |
| Products → orders | `ProductID` | One-to-many |
| Calendar → orders | `Date` = `New Date` | One-to-many |

**Measures used in the PivotTables:** Total Sales (Quantity × Price) and Total Quantity.

## Key Figures

| KPI | Value |
|---|---|
| Customers | 300 |
| Units sold | 108,203 |
| Total sales | $101,244,955 |
| Average sales per order | $5,164 |

| Category | Units | Share of units | Sales | Share of sales |
|---|---|---|---|---|
| Books | 21,415 | 19.8% | $17,210,430 | 17.0% |
| Clothing | 21,909 | 20.2% | $29,142,717 | 28.8% |
| Electronics | 21,795 | 20.1% | $19,027,964 | 18.8% |
| Furniture | 21,731 | 20.1% | $10,445,482 | 10.3% |
| Toys | 21,353 | 19.7% | ≈ $25,418,362 | ≈ 25.1% |

*Toys sales are derived as total sales minus the other four categories.*

## Key Findings

- **Volume is flat; revenue is not.** Each category sells roughly 20% of total units, and monthly volume stays within about 8,900–9,160 units. Differences in revenue come from **price and product mix**, not from selling more units.
- **Clothing and Toys generate about 54% of revenue** with about 40% of units, while **Furniture generates about 10% of revenue** with about 20% of units.
- **Regional customer base is balanced:** 56–62 customers per region.
- Demand is **stable across the year**, with no strong seasonality visible in unit volumes.

## Repository Structure

```
retail-insights-demand-analytics/
├── README.md
├── Power Query.xlsx          # Final workbook (model + pivots + dashboard)
├── data/
│   ├── Products.csv
│   ├── Customers.csv
│   └── orders/               # Monthly order files
├── power-query/
│   ├── orders.m
│   ├── Products.m
│   └── Customers.m
└── images/
    ├── dashboard.png
    ├── data-model.png
    └── pivot-tables.png
```

## How to Use

1. Download the workbook and open it in **Excel for Windows** (Power Pivot is not available in Excel for Mac or Excel Online).
2. If you want to refresh the data: go to **Data → Queries & Connections**, update the source paths (see the note below), then **Refresh All**.
3. Use the slicers (Region, Category, First Name) and the Date timeline to filter the whole dashboard.
4. To inspect the model, open **Power Pivot → Manage → Diagram View**.

> **Source paths:** The queries point to local folders. Change the file and folder paths to your own location, or convert them into a Power Query parameter.

## Limitations & Possible Improvements

- The dataset is synthetic; prices and volumes are uniform, so business conclusions should not be generalised.
- Orders cover a single year (2024), so year-over-year comparison is not possible.
- `Number.RoundUp` on prices increases every price with decimals; use `Number.Round` if exact revenue is required.
- Deduplicating on `OrderID` assumes IDs are unique across all monthly files.
- Possible additions: profit margin, customer-level analysis (top customers, repeat purchases), and a DAX-based YoY / MoM growth measure.

## Author

**Verena**
professional Financial Analyst with a focus on reporting, reconciliation and data analysis.
LinkedIn Profile: 
www.linkedin.com/in/
verena-farag-bb0476217

Feedback and suggestions are welcome.

