# Product_Sales_data_Power_BI-SQL
Product sales data analysis by using Power BI and postgreSQL 
# 📊 Product Sales Analytics — Power BI + PostgreSQL Mini Project

A complete end-to-end data analytics project covering multi-year sales data (2014–2017) across European markets. Raw Excel/CSV files are modelled in **PostgreSQL**, then visualized in **Microsoft Power BI** through a star-schema data model.

---

## 📁 Project Structure

```
MiniProject/
│
├── data/
│   ├── raw/
│   │   ├── Categories.xlsx          # Product category lookup
│   │   ├── SubCategories.xlsx       # Product sub-category lookup
│   │   ├── SalesRep.xlsx            # Sales representative master
│   │   ├── Geography.xlsx           # Country & city reference
│   │   ├── Sales_2014.csv           # Transactional sales data – 2014
│   │   ├── sales_2015.csv           # Transactional sales data – 2015
│   │   ├── sales_2016.csv           # Transactional sales data – 2016
│   │   └── sales_2017.csv           # Transactional sales data – 2017
│   └── sql/
│       └── ProjectDatabaseQueries.txt   # DDL + seed SQL for Products table
│
├── powerbi/
│   └── SalesDashboard.pbix          # Power BI report file
│
└── README.md
```

---

## 🗄️ Data Sources & Schema

### Dimension Tables (from Excel files)

#### `Categories`
| Column | Type | Description |
|--------|------|-------------|
| CategoryKey | INT | Primary key |
| Category | VARCHAR | Category name (`Special`, `General`) |

#### `SubCategories`
| Column | Type | Description |
|--------|------|-------------|
| SubCategoryKey | INT | Primary key |
| CategoryKey | INT | FK → Categories |
| SubCategory Name | VARCHAR | Sub-category name (`Extra`, `Regular`, `Micro`, `Unique`) |

#### `SalesRep`
| Column | Type | Description |
|--------|------|-------------|
| SalesRepID | VARCHAR | Primary key (`ID - 1` … `ID - 7`) |
| Sales Rep Name | VARCHAR | Full name of sales representative |

> **7 sales reps** across the team: Jan Novotny, John White, Ellen Woody, and others.

#### `Geography`
| Column | Type | Description |
|--------|------|-------------|
| Country | VARCHAR | Country name |
| Town | VARCHAR | City name |
| Wikipedia | URL | Wikipedia link for the city |

> Covers **7 cities** across Czech Republic, Denmark, Germany, Netherlands, and Spain.

#### `Products` *(loaded via PostgreSQL)*
| Column | Type | Description |
|--------|------|-------------|
| ProductID | INT | Product identifier |
| SubCategoryKey | INT | FK → SubCategories |
| Color | VARCHAR(50) | Product color |
| ProductName | VARCHAR(100) | Product name |
| RetailPrice | NUMERIC(10,2) | Selling price |
| StandardCost | NUMERIC(10,2) | Cost of goods |

> **Note:** The raw insert data includes duplicate rows (ProductID 7 and 10 appear twice). This is intentional for data-cleaning practice — deduplicate using `DISTINCT` or `ROW_NUMBER()` as needed.

---

### Fact Table — Sales (CSV files, 2014–2017)

All four CSV files share the same schema and are **unioned** in Power BI or PostgreSQL:

| Column | Type | Description |
|--------|------|-------------|
| fSalesPrimaryKey | INT | Unique transaction key |
| ProductID | INT | FK → Products |
| SalesRepID | INT | FK → SalesRep |
| Location | VARCHAR | `Country;City` format (e.g. `Germany;Frankfurt`) |
| Date | VARCHAR | Transaction date (DD.MM.YYYY) |
| Units | INT | Units sold |
| PercentOfStandardCost | DECIMAL | Cost ratio (e.g. `0.954`) |
| RevenueDiscount | DECIMAL | Discount applied to revenue (e.g. `0.55`) |

| Year | Rows |
|------|------|
| 2014 | 12,248 |
| 2015 | 12,500 |
| 2016 | 12,194 |
| 2017 | 17,588 |
| **Total** | **54,530** |

---

## 🐘 PostgreSQL Setup (pgAdmin 4)

### 1. Create the Database
```sql
CREATE DATABASE "MiniProject";
```

### 2. Create & Seed the Products Table
```sql
DROP TABLE IF EXISTS Products;

CREATE TABLE Products (
    ProductID      INT,
    SubCategoryKey INT,
    Color          VARCHAR(50),
    ProductName    VARCHAR(100),
    RetailPrice    NUMERIC(10,2),
    StandardCost   NUMERIC(10,2)
);

INSERT INTO Products (ProductID, SubCategoryKey, Color, ProductName, RetailPrice, StandardCost)
VALUES
(1,  3, 'Red',             'Alder',      23.95, 7.55),
(2,  2, 'Blue',            'Linder',     23.95, 7.55),
(3,  2, 'Green',           'Magnum',     23.95, 7.55),
(4,  1, 'Red',             'Quad',       43.95, 13.75),
(5,  1, 'Blue',            'Black Monk', 43.95, 13.75),
(6,  4, 'Green',           'Quad',       43.95, 13.75),
(7,  1, 'Red',             'Bing',       26.95, 8.25),
(8,  3, 'Blue',            'VanHelen',   26.95, 8.25),
(9,  1, 'Green',           'Magnum',     26.95, 8.25),
(10, 1, 'Florescent Pink', 'Carlota',    29.95, 9.15),
(11, 4, 'Florescent Blue', 'Carlota',    29.95, 9.15),
-- Duplicate rows below — for data-cleaning exercise
(7,  1, 'Red',             'Bing',       26.95, 8.25),
(10, 1, 'Florescent Pink', 'Carlota',    29.95, 9.15);
```


### 3. Verify the Data
```sql
SELECT * FROM Products ORDER BY ProductID;
-- Expected: 13 rows total (11 unique products + 2 duplicate rows if not cleaned)
```

---

## 📊 Power BI Data Model

The report uses a **star schema** connecting the fact table to all dimension tables:

```
                    ┌─────────────┐
                    │  Categories │
                    └──────┬──────┘
                           │ CategoryKey
                    ┌──────┴──────────┐
                    │  SubCategories  │
                    └──────┬──────────┘
                           │ SubCategoryKey
┌──────────┐    ┌──────────┴──────────┐    ┌────────────┐
│ SalesRep │────│   Products (from    │    │ Geography  │
│          │    │   PostgreSQL)       │    │            │
└──────────┘    └──────────┬──────────┘    └─────┬──────┘
      │ SalesRepID          │ ProductID           │ Location
      │              ┌──────┴──────────┐          │
      └──────────────│  Sales Fact     │──────────┘
                     │ (2014–2017 CSV) │
                     └─────────────────┘
```

### Key Relationships

| From | Key | To |
|------|-----|----|
| Sales | `ProductID` | Products |
| Sales | `SalesRepID` | SalesRep |
| Sales | `Location` | Geography |
| Products | `SubCategoryKey` | SubCategories |
| SubCategories | `CategoryKey` | Categories |

---

## 📐 Key DAX Measures (suggested)

```dax
-- Total Revenue
Total Revenue = 
SUMX(Sales, Sales[Units] * RELATED(Products[RetailPrice]) * (1 - Sales[RevenueDiscount]))

-- Total Cost
Total Cost = 
SUMX(Sales, Sales[Units] * RELATED(Products[StandardCost]) * Sales[PercentOfStandardCost])

-- Gross Profit
Gross Profit = [Total Revenue] - [Total Cost]

-- Gross Profit Margin %
GP Margin % = DIVIDE([Gross Profit], [Total Revenue], 0)

-- YoY Revenue Growth
YoY Growth % = 
DIVIDE(
    [Total Revenue] - CALCULATE([Total Revenue], SAMEPERIODLASTYEAR('Date'[Date])),
    CALCULATE([Total Revenue], SAMEPERIODLASTYEAR('Date'[Date])),
    0
)
```

---

## 🔌 Connecting Power BI to PostgreSQL

1. Open Power BI Desktop → **Get Data** → **PostgreSQL database**
2. Enter:
   - **Server:** `localhost`
   - **Database:** `MiniProject`
3. Select the `Products` table and load it
4. Import CSV files separately via **Get Data → Text/CSV**
5. Union all four sales CSVs using **Append Queries**

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| **PostgreSQL 15 / pgAdmin 4** | Database creation, Products table, SQL queries |
| **Microsoft Power BI Desktop** | Data modelling, DAX, dashboards |
| **Excel / CSV** | Raw data sources (dimension & fact tables) |
| **SQL** | DDL, DML, data cleaning |

---

## 🚀 How to Run This Project

1. **Clone the repo**
   ```bash
   git clone https://github.com/<your-username>/MiniProject-SalesAnalytics.git
   cd MiniProject-SalesAnalytics
   ```

2. **Set up PostgreSQL**
   - Install PostgreSQL and open pgAdmin 4
   - Run the SQL in `data/sql/MiniProject2DatabaseQueries.txt`

3. **Open Power BI**
   - Open `powerbi/SalesDashboard.pbix`
   - Update the PostgreSQL connection string to your local server
   - Refresh all data sources

4. **Explore the dashboard**
   - Filter by Year, Country, Sales Rep, Category
   - Analyse revenue trends, top products, and regional performance

---

## 📌 Notes & Known Issues

- The `Location` column in sales CSVs uses `Country;City` format — split this in Power Query using **Split Column by Delimiter (`;`)** to join with the Geography table.
- The `Date` column is in `DD.MM.YYYY` format — change the data type to **Date** in Power Query with locale set to a European locale.
- The `SalesRepID` in `SalesRep.xlsx` is stored as `ID - 6` (string), while in sales CSVs it is stored as a plain integer. A transformation step is required to align these keys.
- Products table contains **2 duplicate rows** (ProductIDs 7 and 10) — this is intentional for SQL practice.

---

## 📄 License

This project is created for educational purposes as part of a data analytics mini project.

---

## 🙋 Author

Ajay Tiwari 🎓 Computer Science & Engineering

GitHub: @Ajaytiwari-code
LinkedIn: ajay-tiwari-68a601291
