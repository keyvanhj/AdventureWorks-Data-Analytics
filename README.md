[README_AdventureWorks.md](https://github.com/user-attachments/files/34737287/README_AdventureWorks.md)
# 🚴 Adventure Works Sales Analytics: Power BI Dashboard

An interactive Power BI report that analyses three years of sales for **Adventure Works**, a fictional bicycle manufacturer. It covers sales and profit performance, actual vs budget, product mix, geography and customer demographics, and it is built on a clean data model with a reusable DAX measure library.

> **Dataset:** Adventure Works sample data (`Adventure Works 2020.xlsx`, 9 sheets). All customers, orders and names are fictitious.

---

## Table of Contents

- [Business questions](#business-questions)
- [Key insights](#key-insights)
- [Dashboard pages](#dashboard-pages)
- [Data model](#data-model)
- [Data preparation (Power Query)](#data-preparation-power-query)
- [DAX measures and calculated columns](#dax-measures-and-calculated-columns)
- [Repository structure](#repository-structure)
- [How to open the report](#how-to-open-the-report)
- [Data notes and limitations](#data-notes-and-limitations)
- [Possible improvements](#possible-improvements)
- [Tech stack](#tech-stack)
- [License](#license)
- [Author](#author)

---

## Business questions

1. How much do we sell, how profitable is it, and how is it changing over time?
2. Are we meeting the budget, in total and by product category?
3. Which products, models and product lines drive sales and profit?
4. Which countries and regions contribute the most?
5. Who are our customers (age, income, occupation, family status) and how do they buy?

## Key insights

All figures are computed from the model for the whole period (1 Jul 2016 to 31 Jul 2019) with no slicers applied.

| KPI | Value |
|---|---|
| Total sales | **$29.36 M** |
| Total cost | $17.28 M |
| Total profit | **$12.08 M** |
| Profit margin | **41.1 %** |
| Orders | 27,659 |
| Customers | 18,484 |
| Units sold | 60,398 |
| Average order value | $1,061 |

**What the data says**

- **Growth.** Calendar-year sales rose from $6.53 M (2017) to $9.79 M (2018), +49.9 %. January to July 2019 already reached $9.77 M. Profit margin is stable at about 40 to 42 % every year.
- **Order value is falling while the order count explodes.** Orders grew from 2,677 (2017) to 10,919 (2018) and 13,050 (2019 YTD), but sales grew far less, so the average order value dropped from about **$2,440 to $900 to $750**. The business is selling to more customers at lower ticket sizes.
- **Bikes are the business.** Bikes generate **96.5 %** of sales ($28.3 M, margin 40.6 %). Accessories (2.4 % of sales) have the **highest margin, 62.6 %**, but are small. Clothing is 1.2 % of sales (margin 40.1 %).
- **Road and Mountain dominate.** Road Bikes ($14.5 M) and Mountain Bikes ($10.0 M) are the top subcategories. The top three models (Mountain-200, Road-150, Road-250) are also the top three by profit; Mountain-200 alone brings $7.9 M in sales and $3.6 M in profit.
- **Two markets carry the company.** The United States (32.0 %) and Australia (30.9 %) make up 63 % of sales, followed by the United Kingdom (11.6 %), Germany (9.9 %), France (9.0 %) and Canada (6.7 %). By group: North America $11.4 M, Pacific $9.1 M, Europe $8.9 M.
- **Close to budget.** Actual sales are **98.4 %** of the budget overall ($29.36 M vs $29.84 M). By category: Bikes 98.5 %, Clothing 96.5 %, Accessories 94.8 %.

## Dashboard pages

The report has four pages (three visible pages plus a hidden tooltip page), with a shared navigation bar, slicers and bookmarks.

### 1. Sales Overview
KPI cards (sales, profit, profit margin, orders), monthly sales vs last year (line), **sales vs budget** by month (clustered column), sales by product category (donut), profit by category (donut), sales by subcategory (treemap) and sales by country (bar).
Slicers: calendar year and territory group. Bookmark buttons switch the view between **Total Sales** and **Total Profit**.

### 2. Products
KPI cards (sales, profit margin, units sold, products sold), **Top 10 models by sales** and **Top 10 models by profit**, sales by subcategory, sales by product line (pie) and a detail table by model (orders, sales, profit, margin).
Slicers: category, product line, class, color and year. Buttons switch between sales and profit.

### 3. Customers
KPI cards (customers, sales per customer, orders per customer, average order value), sales and customers by occupation, sales by **income band** and by **age group**, a matrix of customers by age and income, customers by family status, sales by gender and a customer table.
Slicers: occupation, year, family status, home ownership and gender. Buttons switch between the age and income views.

### 4. Tooltip (hidden page)
A custom tooltip page (monthly sales trend, sales and margin) that appears when hovering over charts.

<!-- Add screenshots here, for example: -->
<!-- ![Sales Overview](docs/images/sales_overview.png) -->
<!-- ![Products](docs/images/products.png) -->
<!-- ![Customers](docs/images/customers.png) -->

## Data model

A star schema with one fact table and a snowflaked product dimension, plus a budget fact table.

```text
                         ┌────────────────┐
                         │    Calendar    │
                         └───────┬────────┘
                  OrderDate (active)│ShipDate (inactive)
                                 │
┌───────────┐   ┌─────────────────┴─────────────┐   ┌─────────────┐
│ Customers ├───┤         Sales(Fact)           ├───┤  Territory  │
└───────────┘   └─────────────────┬─────────────┘   └─────────────┘
                                  │ ProductKey
                            ┌─────┴─────┐
                            │ Products  │
                            └─────┬─────┘
                                  │
                      ┌───────────┴───────────┐
                      │ dimProductSubCategory │
                      └───────────┬───────────┘
                                  │
                      ┌───────────┴───────────┐        ┌──────────┐      ┌──────────────┐
                      │  dimProductCategory   ├────────┤  Budget  ├──────┤ BudgetPeriod │
                      └───────────────────────┘        └──────────┘      └──────────────┘
```

| From (many) | To (one) | Active | Note |
|---|---|---|---|
| `Sales(Fact)[OrderDate]` | `Calendar[Date]` | ✅ | Main time relationship |
| `Sales(Fact)[ShipDate]` | `Calendar[Date]` | ❌ | Used on demand with `USERELATIONSHIP` |
| `Sales(Fact)[ProductKey]` | `Products[ProductKey]` | ✅ | |
| `Sales(Fact)[CustomerKey]` | `Customers[CustomerKey]` | ✅ | |
| `Sales(Fact)[SalesTerritoryKey]` | `Territory[TerritoryKey]` | ✅ | |
| `Products[ProductSubcategoryKey]` | `dimProductSubCategory[ProductSubcategoryKey]` | ✅ | |
| `dimProductSubCategory[ProductCategoryKey]` | `dimProductCategory[ProductCategoryKey]` | ✅ | |
| `Budget[Category]` | `dimProductCategory[EnglishProductCategoryName]` | ✅ | Budget is defined per category |
| `Budget[Period]` | `BudgetPeriod[Period]` | ✅ | `Period` = year and month, e.g. `201607` |

Row counts: Sales 60,398 lines in 27,659 orders · Customers 18,484 · Products 397 · Territories 10 · Calendar 1,126 days · Budget 62 rows (category by month) over 37 months.

**Fiscal year:** the fiscal year starts on **1 July** (ends 30 June). `Calendar[Fiscal Year]` and the YTD measure follow this.

## Data preparation (Power Query)

Each sheet of the Excel workbook is loaded as its own query. Main steps:

| Table | Transformations |
|---|---|
| `Sales(Fact)` | Typed columns (dates, currency, percentage), removed `OrderDate Key` and `RegionMonthID`, added `Profit` column |
| `Customers` | Removed address and phone columns; replaced codes (`M/S` → Married/Single, `M/F` → Male/Female, house owner `1/0` → YES/NO); trimmed text |
| `Products` | Removed redundant columns; trimmed text; replaced nulls with `NA` (class, size, color, product line, size range) and `Old Version` (status); expanded codes (`M/R/T/S` → Mountain/Road/Touring/Standard, `H/M/L` → High/Medium/Low) |
| `Calendar` | Typed columns; removed unused columns; limited to dates up to **31 Jul 2019** (end of the data) |
| `Territory` | Renamed `Territory Key` to `TerritoryKey`; excluded the non-sales territory key 11 |
| `dimProductCategory`, `dimProductSubCategory` | Removed Spanish and French name columns and alternate keys |
| `Budget`, `BudgetPeriod` | Typed columns (`Budget` as currency) |

**Integrity checks (all pass):** every sales row matches a date, product, customer and territory; there are no orphan keys; every product has a category.

## DAX measures and calculated columns

Measures are stored in a dedicated table and follow a naming convention: `$` = currency, `#` = count, `%` = ratio.

| Group | Measure | Definition |
|---|---|---|
| **Core** | `$ Total Sales` | `SUM(SalesAmount)` |
| | `$ Total Cost` | `SUM(TotalProductCost)` |
| | `$ Total Profit` | `[$ Total Sales] - [$ Total Cost]` |
| | `% Profit Margin` | `DIVIDE([$ Total Profit], [$ Total Sales])` |
| | `# Total Orders` | `DISTINCTCOUNT(SalesOrderNumber)` |
| | `# Total Quantity` | `SUM(OrderQuantity)` |
| | `# Products Sold` | Distinct product names with sales greater than 0 |
| **Customers** | `# Total Customers` | `DISTINCTCOUNT(CustomerKey)` |
| | `$ Avg Order Value` | `DIVIDE([$ Total Sales], [# Total Orders])` |
| | `$ Sales Per Customer` | `DIVIDE([$ Total Sales], [# Total Customers])` |
| | `# Orders per Customer` | `DIVIDE([# Total Orders], [# Total Customers])` |
| | `% Customers with Children` | Share of customers with at least one child at home |
| | `$ Profit per Unit` | `DIVIDE([$ Total Profit], [# Total Quantity])` |
| **Time intelligence** | `$ Sales LY` | `CALCULATE([$ Total Sales], SAMEPERIODLASTYEAR(Calendar[Date]))` |
| | `% Sales YoY` | `DIVIDE([$ Total Sales] - [$ Sales LY], [$ Sales LY])` |
| | `$ Sales FYTD` | `TOTALYTD([$ Total Sales], Calendar[Date], "6/30")` (fiscal year to date) |
| | `$ Sales by Ship Date` | Sales using the inactive `ShipDate` relationship (`USERELATIONSHIP`) |
| **Share** | `% Sales Share` | `DIVIDE([$ Total Sales], CALCULATE([$ Total Sales], ALLSELECTED()))` |
| **Budget** | `$ Budget Amount` | Budget aligned with the selected periods using `TREATAS` |
| | `% Sales vs Budget` | `DIVIDE([$ Total Sales], [$ Budget Amount])` |
| | `$ Budget Variance` | `[$ Total Sales] - [$ Budget Amount]` |

**Calculated columns**

| Table | Column | Purpose |
|---|---|---|
| `Customers` | `Age` | Age on 31 Jul 2019 (the last date in the data) |
| `Customers` | `Age Group` and `Age Group Sort` | Under 30, 30-39, 40-49, 50-59, 60+ (with a sort order column) |
| `Customers` | `Income Band` and `Income Band Sort` | Low (<40k), Medium (40k-80k), High (80k-120k), Very High (120k+) |
| `Customers` | `Family Status` | With children / No children |
| `Products` | `Unit Margin %` | `(ListPrice - StandardCost) / ListPrice` |
| `Calendar` | `Month Year` | Label such as `Jul 2018` for axes |

## Repository structure

```text
.
├── README.md
├── dashboard/
│   └── Adventure_Works_Sales_Dashboard.pbix
├── data/
│   └── Adventure Works 2020.xlsx            # see the note on data and license below
└── docs/
    └── images/                              # dashboard screenshots
```

## How to open the report

1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (Windows).
2. Open `dashboard/Adventure_Works_Sales_Dashboard.pbix`.
3. The queries read the Excel workbook from a local path, so point them to your copy:
   **Home → Transform data → Data source settings → Change Source…** and select `data/Adventure Works 2020.xlsx`.
4. Click **Close & Apply**, then **Refresh**.

## Data notes and limitations

- **The last month is incomplete.** July 2019 contains only $50.8 K of sales (2,209 lines, mostly low-value items) vs $1.95 M in June 2019. The monthly trend therefore drops sharply at the end, and year-over-year values for July 2019 should not be interpreted.
- **2019 is a partial year** (January to July), and 2016 starts in July. Compare full calendar years (2017 vs 2018) or use fiscal years (FY2017 to FY2019).
- **Every order line has a quantity of 1** (60,398 lines and 60,398 units), so units and line count are identical, and `Profit` (sales minus unit cost) equals sales minus total cost.
- **Age is calculated at a fixed date** (31 Jul 2019), not at today's date.
- **Fictitious data.** Names, birth dates and orders are sample data and do not describe real people.
- **Product mix is very concentrated:** bikes are 96.5 % of sales, so category-level conclusions about accessories and clothing rest on small amounts.

## Possible improvements

- Replace the hard-coded file path with a Power Query parameter
- Add a year-over-year and budget-variance view by month and category (decomposition tree)
- Add customer cohort or retention analysis using `DateFirstPurchase`
- Investigate why average order value fell from about $2,440 to $750 (product mix, customer mix or pricing)
- Add row-level security by sales territory
- Publish to Power BI Service with a scheduled refresh

## Tech stack

- **Power BI Desktop:** report design, bookmarks, page navigator, tooltip page, slicers
- **Power Query (M):** data cleaning and shaping from Excel
- **DAX:** measures, time intelligence, calculated columns
- **Data modeling:** star schema with a snowflaked product dimension and a budget fact table



> The Adventure Works sample data is provided by Microsoft for learning purposes. Check the terms of the source of `Adventure Works 2020.xlsx` before redistributing it in your repository: `[add source and license of the workbook here]`.

## Author

**`Keyvan Hajizadeh`**

If you find this project useful, feel free to ⭐ the repository.
