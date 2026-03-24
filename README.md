# 🛒 Sales Analytics Dashboard — Power BI

<p align="center">
  <img src="https://img.shields.io/badge/Tool-Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black"/>
  <img src="https://img.shields.io/badge/Domain-Sales%20%26%20Retail-C8A97E?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Dashboards-3-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Model-Star%20Schema-blueviolet?style=for-the-badge"/>
</p>

---

## 📌 Project Overview

A fully interactive **Sales Analytics Dashboard** built in **Power BI**, powered by a clean **Star Schema data model** connecting 6 source files (Excel & CSV) merged and cleaned into a single structured dataset.

The project delivers a **360° sales intelligence solution** covering customer behavior, employee performance, and inventory analysis — with custom DAX measures, dynamic tooltips, and cross-page navigation across **3 professional dashboards**.

### 📁 Data Sources
The project started with **6 raw files** (Excel & CSV):
- `Customers` data file
- `Employees` data file
- `Stock Items` data file
- `Sales / Transactions` data file
- `Cities` reference file
- `Dates` reference file

All 6 files were **merged into a single Excel workbook**, cleaned thoroughly, then connected to Power BI as the single source of truth.

---

## 🗂️ Data Model — Star Schema

The model follows a clean **Star Schema** design with `FactSale` at the center, connected to 4 dimension tables:

```
DimCustomer ──┐
DimEmployee ──┤
DimCity      ──┼──► FactSale ◄── DimDate
DimStockItem ──┘
```

### Tables & Relationships

| Table | Type | Key Column | Description |
|-------|------|-----------|-------------|
| `FactSale` | Fact | Sale Key | All sales transactions — quantity, profit, tax, unit price |
| `DimCustomer` | Dimension | Customer Key | Customer info, billing group, branch, credit limit |
| `DimEmployee` | Dimension | Employee Key | Employee details, salesperson flag, validity dates |
| `DimCity` | Dimension | City Key | City, state/province, population |
| `DimStockItem` | Dimension | Stock Item Key | Product details, brand, package, weight, price |
| `DimDate` | Dimension | Date | Full date hierarchy — calendar, fiscal year, month, week |
| `_Measures` | Measures Table | — | Isolated table for all DAX measures |

### Relationship Details
- All relationships are **One-to-Many (1:*)** from dimension to fact
- `DimDate` connects to `FactSale` via both **Invoice Date Key** and **Delivery Date Key** (active/inactive relationships)
- Cross-filter direction: **Single** for all relationships
- No bi-directional filtering to maintain model performance

---

## 🧹 Data Cleaning

Raw data from 6 files was merged into one Excel workbook, then cleaned through a rigorous process before loading into Power BI:

### In Excel (Pre-Load Cleaning):
1. **Merged 6 Files into 1** — Combined all Excel and CSV files into a single structured workbook using consistent column naming
2. **Removed Duplicates** — Eliminated duplicate rows across the Sales, Customer, and Employee sheets
3. **Handled Missing Values** — Filled nulls in key columns (Customer Key, City Key, Stock Item Key); removed rows with unresolvable missing foreign keys
4. **Standardized Text Fields** — Applied consistent casing and formatting to Customer Name, City, Branch Name, and Product Name columns
5. **Fixed Data Types** — Converted date columns stored as text to proper Date format; ensured all numeric fields (Profit, Quantity, Unit Price) were stored as numbers
6. **Validated Foreign Keys** — Verified that all keys in the fact table existed in their respective dimension tables to avoid broken relationships
7. **Removed Irrelevant Columns** — Dropped columns not needed for analysis to reduce model size
8. **Handled Outliers** — Reviewed abnormal sales values and profit margins for data integrity

### In Power Query (Post-Load Cleaning):
9. **Renamed Columns** — Standardized column names to PascalCase for consistency across all tables
10. **Changed Data Types** — Enforced correct types for each column in Power Query Editor
11. **Created Calculated Columns** — Added `Is Salesperson` flag logic and `Valid From / Valid To` employee validity columns
12. **Filtered Unnecessary Rows** — Removed test transactions and null-key rows from FactSale
13. **Sorted & Indexed Tables** — Applied proper sorting on DimDate by Date column

---

## ⚙️ DAX Measures

All measures are stored in a dedicated `_Measures` table for clean model organization.

### 💰 Sales & Profit Measures
```dax
-- Total Sales
Total Sales = SUM(FactSale[Total Including Tax])

-- Total Profit
Total Profit = SUM(FactSale[Profit])

-- Profit Margin %
Profit Margin % = DIVIDE([Total Profit], [Total Sales], 0)

-- Average Profit
Average Profit = AVERAGE(FactSale[Profit])

-- Avg Sales per Transaction
Avg Sales per Transaction = DIVIDE([Total Sales], [Sales Transactions], 0)
```

### 📦 Items & Inventory Measures
```dax
-- Total Items Sold
Total Items Sold = SUM(FactSale[Quantity])

-- Avg Items per Sale
Avg Items per Sale = DIVIDE([Total Items Sold], [Sales Transactions], 0)

-- Total Stock Items
Total Stock Items = DISTINCTCOUNT(DimStockItem[Stock Item Key])

-- Total Stock Value
Total Stock Value = SUMX(DimStockItem, DimStockItem[Unit Price] * DimStockItem[Quantity Per Outer])

-- AVG Lead Time Days
AVG Lead Time Days = AVERAGE(DimStockItem[Lead Time Days])

-- AVG Typical Weight Per Unit
AVG Typical Weight Per Unit = AVERAGE(DimStockItem[Typical Weight Per Unit])
```

### 👥 Customer & Employee Measures
```dax
-- Total Customers
Total Customers = DISTINCTCOUNT(FactSale[Customer Key])

-- Sales Transactions
Sales Transactions = COUNTROWS(FactSale)

-- AVG Recommended Retail Price
AVG Recommended Retail Price = AVERAGE(DimStockItem[Recommended Retail Price])

-- AVG Unit Price
AVG Unit Price = AVERAGE(DimStockItem[Unit Price])
```

### 🧮 Time Intelligence Measures
```dax
-- Total Sales YTD
Total Sales YTD =
TOTALYTD([Total Sales], DimDate[Date])

-- Total Profit YTD
Total Profit YTD =
TOTALYTD([Total Profit], DimDate[Date])

-- Sales vs Previous Year
Sales PY =
CALCULATE([Total Sales], SAMEPERIODLASTYEAR(DimDate[Date]))

-- Sales Growth %
Sales Growth % =
DIVIDE([Total Sales] - [Sales PY], [Sales PY], 0)
```

---

## 📊 Dashboards

### 🏷️ Dashboard 1 — Customers Dashboard

Focused on customer-level sales performance and geographic distribution.

**KPIs:**
| Metric | Value |
|--------|-------|
| Total Sales | 16.94M |
| Total Profit | 9.92M |
| Total Items Sold | 1M |
| Average Profit | 375.95 |

**Visuals:**
- 📈 Total Sales & Total Profit by Calendar Year (Line Chart — 2013 to 2016)
- 📋 Total Profit by Customer (Bar/Table visual)
- 🍩 Total Sales by Customer Group (Donut Chart — Wingtip Toys 25.42%, Tailspin Toys 40.53%, Blank 34.05%)
- 🗺️ Total Customers & Total Sales by City (Azure Map with bubble size = sales volume)
- 🎛️ Filters: Billing and Branch Group / Fiscal Year Label / Branch Name

**Custom Tooltip:**
Hovering over the map reveals a custom tooltip card showing:
- Total Sales
- Total Profit
- Profit Margin %

---

### 👔 Dashboard 2 — Employees Dashboard

Focused on sales rep performance, productivity, and geographic sales analysis.

**KPIs:**
| Metric | Value |
|--------|-------|
| Sales Transactions | 26K |
| Profit Margin % | 58.59% |
| Avg Items per Sale | 38.97 |
| Avg Sales per Transaction | 641.64 |

**Visuals:**
- 📊 Total Items Sold by Preferred Name (Bar Chart — Top performers: Hudson 200K, Kayla 108K, Taj 106K)
- 🍩 Profit Margin % by Is Salesperson (Donut Chart — Top Salesperson segment)
- 📋 Employees Tracking Table (Employee | Total Sales | Profit Margin % | Avg Sales per Transaction)
- 🗺️ Sales Analysis (Decomposition Tree-style visual — City × Preferred Name breakdown)
- 🎛️ Filters: Employee / City / Calendar Year / Branch Name

---

### 📦 Dashboard 3 — Stock Items Dashboard

Focused on inventory management, product packaging, and stock performance.

**KPIs:**
| Metric | Value |
|--------|-------|
| Total Stock Items | 671 |
| Total Stock Value | 90.63K |
| AVG Lead Time Days | 12.34 |
| AVG Typical Weight Per Unit | 1.93 |

**Visuals:**
- 📊 Total Stock Items by Selling Package (Bar Chart — Each: 630, Packet: 17, Bag: 16, Pair: 8)
- 📊 Total Stock Value by Buying Package (Bar Chart — Each: 68K, Carton: 20K, Packet: 3K)
- 🍩 Total Stock Items by Barcode (Donut Chart — Not Available: 97.62%)
- 🍩 Total Stock Items by Brand (Donut Chart — No Brand: 90.01%, Northwind: 9.99%)
- 📋 Stock Items Tracking Table (Stock Item | Total Sales | Profit Margin % | Total Items | Unit Price | AVG Typical Weight | AVG Lead Time)
- 🎛️ Filters: Brand / Calendar Year / Is Chiller Stock / Barcode

**Custom Tooltip on Table:**
Hovering over any stock item row reveals:
- Total Sales
- Total Profit
- Profit Margin %

---

## 🧭 Navigation & UX

- **Top navigation bar** on every dashboard with buttons: `Customers` | `Employees` | `Stock Items`
- Active page is **bold** in the nav bar for clear orientation
- Consistent **warm beige & brown** color theme across all 3 dashboards
- Custom **Sales Logo** icon in the top-left corner
- Matching **thematic icons** per dashboard (customers group icon, handshake for employees, factory/truck for stock)
- **Custom Tooltip Page** — a separate Power BI tooltip report page showing Sales, Profit, and Profit Margin % on hover

---

## Dashboard Screenshots (Click to enlarge) :
<img src="https://github.com/Mohamed-Nofal-DataAnalysis/Sales-Analytics-Dashboard-PowerBI/blob/main/Customers.png">
<img src="https://github.com/Mohamed-Nofal-DataAnalysis/Sales-Analytics-Dashboard-PowerBI/blob/main/Employees.png">
<img src="https://github.com/Mohamed-Nofal-DataAnalysis/Sales-Analytics-Dashboard-PowerBI/blob/main/Stock%20Items.png">
<img src="https://github.com/Mohamed-Nofal-DataAnalysis/Sales-Analytics-Dashboard-PowerBI/blob/main/Tool%20tip.png">
<img src="https://github.com/Mohamed-Nofal-DataAnalysis/Sales-Analytics-Dashboard-PowerBI/blob/main/Model.png">

---

## 📁 File Structure

```
Sales-Analytics-Dashboard-PowerBI/
│
├── 📊 Sales_Dashboard.pbix          # Main Power BI file
├── 📂 Data/
│   └── 📗 Sales_Data_Merged.xlsx    # Merged & cleaned Excel source (6 files combined)
├── 📄 README.md                     # Project documentation
└── 📸 Screenshots/
    ├── customers_dashboard.png
    ├── employees_dashboard.png
    ├── stock_items_dashboard.png
    ├── data_model.png
    └── tooltip.png
```

---

## 🚀 How to Use

1. **Download** `Sales_Dashboard.pbix`
2. **Open** in Power BI Desktop (latest version recommended)
3. If prompted, update the data source path to point to `Sales_Data_Merged.xlsx`
4. Click **Refresh** to reload data
5. Use the **top navigation buttons** to switch between the 3 dashboards
6. Use the **dropdown slicers** on each page to filter by Branch, Year, Employee, etc.
7. **Hover** over map bubbles or table rows to see the custom tooltip

---

## 🛠️ Tools & Technologies

| Tool | Usage |
|------|-------|
| Microsoft Power BI Desktop | Dashboard development, DAX, data modeling |
| Power Query (M Language) | Data transformation and cleaning post-load |
| Microsoft Excel | Pre-load data merging and cleaning (6 → 1 file) |
| DAX | All KPI calculations, time intelligence, ratios |
| Azure Maps (Power BI) | Geographic sales and customer visualization |
| Star Schema Design | Data model architecture |

---

## 🔎 Project Insights

### 💰 Sales & Profit
- Total sales reached **$16.94M** with a healthy **profit margin of 58.59%** — indicating strong pricing discipline
- **Tailspin Toys** is the largest customer group at **40.53%** of total sales, followed by Blank (34.05%) and Wingtip Toys (25.42%)
- Sales and profit trended **steadily from 2013 to 2015** before a visible dip in **2016** — worth investigating for churn or market factors

### 👔 Employee Performance
- **Hudson** leads in items sold (**200K**) — nearly double the second-highest performer (Kayla at 108K)
- **Sophia Hinton** tops total sales revenue at **$1,889,481.75** with a profit margin of 57.58%
- **Profit margin is consistent** across top salespeople (57–58%) — suggesting standardized pricing with little discounting variance
- The city of **Akhiok** generates the highest city-level sales at **$463,274.55**

### 📦 Inventory
- **671 stock items** with a total stock value of **$90.63K** and an average lead time of **12.34 days**
- **"Each"** is the dominant selling package at **630 items (93.9%)** — far ahead of Packet, Bag, and Pair
- **90.01% of items have no brand** — a significant unbranded inventory that may affect perceived value
- **97.62% of items lack a barcode** — a potential operational gap for inventory tracking and retail scanning

---

## ⭐ Final Conclusion

This **Sales Analytics Dashboard** showcases the full data analytics pipeline — from raw, scattered files all the way to polished, interactive business intelligence — using Power BI at a professional level.

### What This Project Achieves:
- ✅ Consolidates **6 raw data sources** into a single clean, structured dataset
- ✅ Implements a proper **Star Schema** data model following BI best practices
- ✅ Delivers **3 focused dashboards** serving different business stakeholders (sales, HR, inventory)
- ✅ Uses **advanced DAX** for dynamic KPIs, time intelligence, and ratio calculations
- ✅ Enhances UX with **custom tooltips**, cross-page navigation, and consistent visual design

### Key Takeaways for Business:
> 📌 With a **58.59% profit margin**, the business is operationally healthy — but the 2016 sales dip signals a need for customer retention analysis.

> 📌 **Hudson's dominance** in items sold vs. Sophia's dominance in revenue suggests a difference in product mix — high-volume low-value vs. low-volume high-value selling strategies.

> 📌 The **97.62% barcode gap** in stock items is an operational red flag that could impact warehouse efficiency and retail readiness.

> 📌 **90% unbranded inventory** is a strategic opportunity — investing in branding could significantly improve perceived product value and customer loyalty.

### Skills Demonstrated:
| Skill | Application |
|-------|-------------|
| Data Merging | Combined 6 Excel/CSV files into 1 structured workbook |
| Data Cleaning | Duplicates, nulls, type errors, foreign key validation |
| Power Query | Post-load transformation, column renaming, filtering |
| Data Modeling | Star Schema design with 1:* relationships |
| DAX | 15+ measures — aggregations, ratios, time intelligence |
| Data Visualization | 3 dashboards, 15+ charts, maps, tables |
| UX Design | Navigation, tooltips, consistent branding |
| Business Analysis | Insight extraction across sales, HR, and inventory domains |

---

> 💡 *"A good data model is invisible — users only see the insights, not the complexity behind them."*
> This project was built with that principle at its core.

---

## 👨‍💼 Author
Mohamed Nofal
Data & Business Analyst  
Transforming raw data into actionable business insights.
> Built with ❤️ using Power BI
> Feel free to ⭐ the repo if you found it useful!

