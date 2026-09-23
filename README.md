# SQL Data Warehouse & Analytics Project

An end-to-end **SQL Server data warehouse project** that integrates CRM and ERP source data, transforms it through Bronze, Silver, and Gold layers, validates data quality, and produces business-ready analytical models.

The project demonstrates how raw operational data can be transformed into a structured analytics platform using:

- **ETL / ELT workflows** with stored procedures
- **Bronze → Silver → Gold** architecture
- Data cleaning and standardization
- Duplicate handling and historical data logic
- Data-quality validation
- Dimensional modeling and star-schema design
- Customer, product, and sales fact modeling
- Reusable Gold-layer views for analytics

## What This Project Does

The warehouse combines customer, product, category, location, and sales data from CRM and ERP source files.

The pipeline follows:

```text
CRM / ERP Sources
        ↓
Bronze — Raw Data
        ↓
Silver — Cleaned & Standardized Data
        ↓
Gold — Business-Ready Dimensions & Fact
        ↓
Analytics / Reporting
```

The final Gold layer provides:

- `gold.dim_customers`
- `gold.dim_products`
- `gold.fact_sales`

These models support analysis of sales, customers, products, countries, and other business dimensions.

## Architecture

### Bronze Layer

Stores raw source data with minimal transformation.

- Creates raw warehouse tables
- Loads CRM and ERP CSV files using `BULK INSERT`
- Refreshes source tables before loading

### Silver Layer

Cleans, standardizes, and transforms Bronze data.

Examples include:

- Trimming text fields
- Standardizing gender and marital-status values
- Handling missing product costs
- Cleaning customer and location keys
- Converting integer date values into SQL dates
- Validating and correcting sales values
- Removing duplicate customer records with `ROW_NUMBER()`
- Deriving product validity periods with `LEAD()`

### Gold Layer

Creates business-ready analytical views:

- `gold.dim_customers`
- `gold.dim_products`
- `gold.fact_sales`

These views form a dimensional model designed for reporting and analysis.

## Data Sources

### CRM

| Source | Description |
|---|---|
| `cust_info.csv` | Customer master data |
| `prd_info.csv` | Product information and product history |
| `sales_details.csv` | Sales transaction data |

### ERP

| Source | Description |
|---|---|
| `CUST_AZ12.csv` | Customer birth date and gender |
| `LOC_A101.csv` | Customer country/location |
| `PX_CAT_G1V2.csv` | Product categories and subcategories |

## Gold Data Model

### `gold.dim_customers`

Customer attributes including customer key, ID, name, country, marital status, gender, birthdate, and creation date.

### `gold.dim_products`

Product attributes including product key, product ID, product name, category, subcategory, cost, product line, and start date.

Historical product versions are filtered so the analytical view represents the current product record.

### `gold.fact_sales`

Transaction-level measures including order number, customer key, product key, order date, shipping date, due date, sales amount, quantity, and price.

## SQL Skills Demonstrated

- Database and schema creation
- DDL and table design
- Stored procedures
- `BULK INSERT`
- `TRUNCATE TABLE`
- `JOIN` operations
- `CASE`
- `TRIM`, `REPLACE`, and `ISNULL`
- Type and date conversion
- `ROW_NUMBER()`
- `LEAD()`
- Window functions
- Data standardization
- Duplicate handling
- Data-quality validation
- Fact and dimension modeling
- Star-schema design
- ETL / ELT workflow design
- `TRY...CATCH` error handling

## Data Quality Checks

### Silver layer

The project checks for:

- Null or duplicate business keys
- Unwanted spaces
- Inconsistent categorical values
- Missing or negative product costs
- Invalid date ranges
- Invalid order, shipping, and due-date relationships
- Sales consistency: `Sales = Quantity × Price`
- Invalid birth dates
- Country standardization

### Gold layer

The project checks for:

- Unique customer surrogate keys
- Unique product surrogate keys
- Connectivity between the fact table and customer/product dimensions

Quality-check scripts:

```text
tests/quality_checks_silver.sql
tests/quality_checks_gold.sql
```

## Project Structure

```text
sql-data-warehouse-project/
│
├── datasets/
│   ├── source_crm/
│   │   ├── cust_info.csv
│   │   ├── prd_info.csv
│   │   └── sales_details.csv
│   │
│   └── source_erp/
│       ├── CUST_AZ12.csv
│       ├── LOC_A101.csv
│       └── PX_CAT_G1V2.csv
│
├── scripts/
│   ├── init_database.sql
│   │
│   ├── bronze/
│   │   ├── ddl_bronze.sql
│   │   └── proc_load_bronze.sql
│   │
│   ├── silver/
│   │   ├── ddl_silver.sql
│   │   └── proc_load_silver.sql
│   │
│   └── gold/
│       └── ddl.gold.sql
│
├── tests/
│   ├── quality_checks_silver.sql
│   └── quality_checks_gold.sql
│
└── LICENSE
```

## How to Run

### Prerequisites

- Microsoft SQL Server
- SQL Server Management Studio (SSMS)

The project uses SQL Server-specific features such as `BULK INSERT`, stored procedures, schemas, and `TRY...CATCH`.

### 1. Clone the repository

```bash
git clone https://github.com/bhaskar-nb/sql-data-warehouse-project.git
cd sql-data-warehouse-project
```

### 2. Create the warehouse

Run:

```text
scripts/init_database.sql
```

This creates the `Datawarehouse` database and the `bronze`, `silver`, and `gold` schemas.

> **Warning:** The initialization script drops and recreates the database if it already exists.

### 3. Create Bronze tables

Run:

```text
scripts/bronze/ddl_bronze.sql
```

### 4. Configure CSV paths

Before loading the Bronze layer, update the machine-specific CSV paths in:

```text
scripts/bronze/proc_load_bronze.sql
```

### 5. Load Bronze data

Create the procedure and execute:

```sql
EXEC bronze.load_bronze;
```

### 6. Create Silver tables

Run:

```text
scripts/silver/ddl_silver.sql
```

### 7. Load and transform Silver data

Create and execute:

```sql
EXEC silver.load_silver;
```

### 8. Create Gold views

Run:

```text
scripts/gold/ddl.gold.sql
```

### 9. Run quality checks

Execute:

```text
tests/quality_checks_silver.sql
tests/quality_checks_gold.sql
```

## Example Analytical Queries

### Total Sales

```sql
SELECT
    SUM(sales_amount) AS total_sales
FROM gold.fact_sales;
```

### Sales by Product Category

```sql
SELECT
    p.category,
    p.subcategory,
    SUM(f.sales_amount) AS total_sales
FROM gold.fact_sales f
JOIN gold.dim_products p
    ON f.product_key = p.product_key
GROUP BY
    p.category,
    p.subcategory
ORDER BY
    total_sales DESC;
```

### Sales by Country

```sql
SELECT
    c.country,
    SUM(f.sales_amount) AS total_sales
FROM gold.fact_sales f
JOIN gold.dim_customers c
    ON f.customer_key = c.customer_key
GROUP BY
    c.country
ORDER BY
    total_sales DESC;
```

### Customer Sales Contribution

```sql
SELECT
    c.customer_key,
    c.first_name,
    c.last_name,
    SUM(f.sales_amount) AS total_sales
FROM gold.fact_sales f
JOIN gold.dim_customers c
    ON f.customer_key = c.customer_key
GROUP BY
    c.customer_key,
    c.first_name,
    c.last_name
ORDER BY
    total_sales DESC;
```

## End-to-End Workflow

```text
Source CSV Files
      ↓
Database Initialization
      ↓
Bronze DDL
      ↓
Bronze Load Procedure
      ↓
Silver DDL
      ↓
Silver ETL / Data Cleaning
      ↓
Silver Quality Checks
      ↓
Gold Dimension + Fact Views
      ↓
Gold Quality Checks
      ↓
Analytics / Reporting
```

## What This Project Demonstrates

This project demonstrates how SQL can be used beyond isolated queries to build a structured analytics data pipeline.

Key capabilities include:

- Integrating CRM and ERP source data
- Building repeatable loading procedures
- Applying data cleaning and standardization rules
- Using window functions for deduplication and historical logic
- Designing analytical dimensions and fact tables
- Implementing data-quality checks
- Creating reusable business-ready views
- Preparing a warehouse for downstream reporting and analysis

## Technologies

- **Database:** Microsoft SQL Server
- **IDE:** SQL Server Management Studio (SSMS)
- **Language:** T-SQL
- **Data Format:** CSV
- **Architecture:** Bronze / Silver / Gold (Medallion-style)
- **Data Model:** Dimensional / Star Schema

## Author

**Bhaskar Nakka**

Computer Science Engineering Graduate | Aspiring Data Analyst

Skills demonstrated:
`SQL` • `T-SQL` • `SQL Server` • `Data Warehousing` • `ETL` • `Data Cleaning` • `Data Modeling` • `Data Quality` • `Window Functions`

## License

This project is available under the MIT License. See [LICENSE](LICENSE) for details.
