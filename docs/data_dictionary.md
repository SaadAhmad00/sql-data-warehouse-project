# Data Dictionary Catalog
**Project:** `sql-data-warehouse-project`  
**Repository:** `SaadAhmad00/sql-data-warehouse-project`  
**Architecture:** Medallion (Bronze → Silver → Gold)  
**Database Platform:** SQL Server (T-SQL)

---

## 1) Overview

This catalog documents:
- Source datasets (CSV files)
- Bronze layer tables (raw ingestion)
- Silver layer tables (cleaned/standardized)
- Gold layer analytical views (star schema style)
- Key data quality/standardization rules implemented in ETL

> Note: This dictionary reflects the current repository scripts and dataset headers.

---

## 2) Source Files (datasets)

## `datasets/cust_info.csv` (CRM Customers)
| Column | Type (source) | Description |
|---|---|---|
| `cst_id` | integer | Customer ID (business identifier). |
| `cst_key` | string | Customer number/key (e.g., `AW00011000`). |
| `cst_firstname` | string | First name (contains spacing inconsistencies in source). |
| `cst_lastname` | string | Last name (contains spacing inconsistencies in source). |
| `cst_marital_status` | string | Marital status code (`M`, `S`, sometimes missing/dirty). |
| `cst_gndr` | string | Gender code (`M`, `F`, sometimes missing/dirty). |
| `cst_create_date` | date | Customer create date. |

## `datasets/prd_info.csv` (CRM Products)
| Column | Type (source) | Description |
|---|---|---|
| `prd_id` | integer | Product ID. |
| `prd_key` | string | Product business key with encoded category prefix. |
| `prd_nm` | string | Product name. |
| `prd_cost` | numeric/integer | Product cost (can be null in source). |
| `prd_line` | string | Product line code (`M`,`R`,`S`,`T`, etc.). |
| `prd_start_dt` | date | Product version/effective start date. |
| `prd_end_dt` | date | Product version end date (nullable in source). |

## `datasets/sales_details.csv` (CRM Sales Transactions)
| Column | Type (source) | Description |
|---|---|---|
| `sls_ord_num` | string | Sales order number. |
| `sls_prd_key` | string | Product key (matches transformed product key in silver/gold). |
| `sls_cust_id` | integer | Customer ID. |
| `sls_order_dt` | int (YYYYMMDD) | Order date in integer format. |
| `sls_ship_dt` | int (YYYYMMDD) | Ship date in integer format. |
| `sls_due_dt` | int (YYYYMMDD) | Due date in integer format. |
| `sls_sales` | numeric/integer | Sales amount. |
| `sls_quantity` | integer | Quantity sold. |
| `sls_price` | numeric/integer | Unit price. |

## `datasets/CUST_AZ12.csv` (ERP Customer Attributes)
| Column | Type (source) | Description |
|---|---|---|
| `CID` | string | Customer identifier (often prefixed, e.g., `NASAW...`). |
| `BDATE` | date | Birth date (future dates can appear in source). |
| `GEN` | string | Gender values (`Male`, `Female`, `M`, `F`, blanks, etc.). |

## `datasets/LOC_A101.csv` (ERP Customer Location)
| Column | Type (source) | Description |
|---|---|---|
| `CID` | string | Customer identifier (e.g., `AW-00011000`). |
| `CNTRY` | string | Country code/name (`US`, `DE`, `United Kingdom`, etc.). |

## `datasets/PX_CAT_G1V2.csv` (ERP Product Category Mapping)
| Column | Type (source) | Description |
|---|---|---|
| `ID` | string | Category ID (e.g., `BI_RB`, `CO_RF`). |
| `CAT` | string | Category name (e.g., Bikes, Components). |
| `SUBCAT` | string | Subcategory name. |
| `MAINTENANCE` | string | Maintenance indicator (`Yes`/`No`). |

---

## 3) Bronze Layer (Raw Tables)

Schema: `bronze`  
Definition script: `scripts/bronze/ddl_bronze.sql`  
Load procedure: `scripts/proc_load_bronze.sql`

## `bronze.crm_cust_info`
| Column | SQL Type | Description |
|---|---|---|
| `cst_id` | `INT` | Customer ID from CRM source. |
| `cst_key` | `NVARCHAR(50)` | Customer key/number from CRM source. |
| `cst_firstname` | `NVARCHAR(50)` | Raw first name. |
| `cst_lastname` | `NVARCHAR(50)` | Raw last name. |
| `cst_material_status` | `NVARCHAR(50)` | Raw marital status code (note spelling in DDL: `material`). |
| `cst_gndr` | `NVARCHAR(50)` | Raw gender code. |
| `cst_create_date` | `DATE` | Raw create date. |

## `bronze.crm_prd_info`
| Column | SQL Type | Description |
|---|---|---|
| `prd_id` | `INT` | Product ID. |
| `prd_key` | `NVARCHAR(50)` | Raw product key. |
| `prd_nm` | `NVARCHAR(50)` | Product name. |
| `prd_cost` | `INT` | Product cost (raw). |
| `prd_line` | `NVARCHAR(50)` | Product line code (raw). |
| `prd_start_dt` | `DATETIME` | Product start date/time (raw). |
| `prd_end_dt` | `DATETIME` | Product end date/time (raw). |

## `bronze.crm_sales_details`
| Column | SQL Type | Description |
|---|---|---|
| `sls_ord_num` | `NVARCHAR(50)` | Sales order number. |
| `sls_prd_key` | `NVARCHAR(50)` | Product key. |
| `sls_cust_id` | `INT` | Customer ID. |
| `sls_order_dt` | `INT` | Order date as YYYYMMDD integer. |
| `sls_ship_dt` | `INT` | Ship date as YYYYMMDD integer. |
| `sls_due_dt` | `INT` | Due date as YYYYMMDD integer. |
| `sls_sales` | `INT` | Sales amount. |
| `sls_quantity` | `INT` | Quantity. |
| `sls_price` | `INT` | Unit price. |

## `bronze.erp_loc_a101`
| Column | SQL Type | Description |
|---|---|---|
| `cid` | `NVARCHAR(50)` | Customer ID from ERP location file. |
| `cntry` | `NVARCHAR(50)` | Raw country code/name. |

## `bronze.erp_cust_az12`
| Column | SQL Type | Description |
|---|---|---|
| `cid` | `NVARCHAR(50)` | Customer ID from ERP customer file. |
| `bdate` | `DATE` | Birth date. |
| `gen` | `NVARCHAR(50)` | Raw gender value. |

## `bronze.erp_px_cat_g1v2`
| Column | SQL Type | Description |
|---|---|---|
| `id` | `NVARCHAR(50)` | Category ID. |
| `cat` | `NVARCHAR(50)` | Category name. |
| `subcat` | `NVARCHAR(50)` | Subcategory name. |
| `maintenance` | `NVARCHAR(50)` | Maintenance flag. |

---

## 4) Silver Layer (Cleansed/Conformed Tables)

Schema: `silver`  
Definition script: `scripts/silver/silver_ddl.sql`  
Load procedure: `scripts/proc_load_silver.sql`

All silver tables include:
- `dwh_create_date DATETIME2 DEFAULT GETDATE()`

## `silver.crm_cust_info`
| Column | SQL Type | Description |
|---|---|---|
| `cst_id` | `INT` | Customer ID (deduplicated latest record). |
| `cst_key` | `NVARCHAR(50)` | Customer number/key. |
| `cst_firstname` | `NVARCHAR(50)` | Trimmed first name. |
| `cst_lastname` | `NVARCHAR(50)` | Trimmed last name. |
| `cst_material_status` | `NVARCHAR(50)` | Standardized marital status (`Single`, `Married`, `n/a`). |
| `cst_gndr` | `NVARCHAR(50)` | Standardized gender (`Male`, `Female`, `n/a`). |
| `cst_create_date` | `DATE` | Customer create date. |
| `dwh_create_date` | `DATETIME2` | Warehouse load timestamp. |

## `silver.crm_prd_info`
| Column | SQL Type | Description |
|---|---|---|
| `prd_id` | `INT` | Product ID. |
| `cat_id` | `NVARCHAR(50)` | Category ID derived from `prd_key` prefix (e.g., `CO_RF`). |
| `prd_key` | `NVARCHAR(50)` | Product key stripped from original composite key. |
| `prd_nm` | `NVARCHAR(50)` | Product name. |
| `prd_cost` | `INT` | Cost with nulls defaulted to `0`. |
| `prd_line` | `NVARCHAR(50)` | Standardized line (`Mountain`, `Road`, `Other Sales`, `Touring`, `n/a`). |
| `prd_start_dt` | `DATE` | Effective start date. |
| `prd_end_dt` | `DATE` | Calculated end date via `LEAD(start)-1 day` (SCD-style). |
| `dwh_create_date` | `DATETIME2` | Warehouse load timestamp. |

## `silver.crm_sales_details`
| Column | SQL Type | Description |
|---|---|---|
| `sls_ord_num` | `NVARCHAR(50)` | Order number. |
| `sls_prd_key` | `NVARCHAR(50)` | Product key. |
| `sls_cust_id` | `INT` | Customer ID. |
| `sls_order_dt` | `DATE` | Parsed order date; invalid/zero values set to NULL. |
| `sls_ship_dt` | `DATE` | Parsed ship date; invalid/zero values set to NULL. |
| `sls_due_dt` | `DATE` | Parsed due date; invalid/zero values set to NULL. |
| `sls_sales` | `INT` | Corrected sales amount if invalid (`qty * abs(price)`). |
| `sls_quantity` | `INT` | Quantity. |
| `sls_price` | `INT` | Corrected unit price if null/<=0 (`sales / qty`). |
| `dwh_create_date` | `DATETIME2` | Warehouse load timestamp. |

## `silver.erp_cust_az12`
| Column | SQL Type | Description |
|---|---|---|
| `cid` | `NVARCHAR(50)` | Normalized customer ID (`NAS` prefix removed when present). |
| `bdate` | `DATE` | Birthdate; future dates set to NULL. |
| `gen` | `NVARCHAR(50)` | Standardized gender (`Male`, `Female`, `n/a`). |
| `dwh_create_date` | `DATETIME2` | Warehouse load timestamp. |

## `silver.erp_loc_a101`
| Column | SQL Type | Description |
|---|---|---|
| `cid` | `NVARCHAR(50)` | Normalized customer ID (`-` removed). |
| `cntry` | `NVARCHAR(50)` | Standardized country (`DE→Germany`, `US/USA→United States`, blank→`n/a`). |
| `dwh_create_date` | `DATETIME2` | Warehouse load timestamp. |

## `silver.erp_px_cat_g1v2`
| Column | SQL Type | Description |
|---|---|---|
| `id` | `NVARCHAR(50)` | Category ID. |
| `cat` | `NVARCHAR(50)` | Category. |
| `subcat` | `NVARCHAR(50)` | Subcategory. |
| `maintenance` | `NVARCHAR(50)` | Maintenance flag. |
| `dwh_create_date` | `DATETIME2` | Warehouse load timestamp. |

---

## 5) Gold Layer (Business Views)

Schema: `gold`  
Definition script: `scripts/gold layer/ddl_gold.sql`

## `gold.dim_customers`
| Column | Type | Description |
|---|---|---|
| `customer_key` | integer | Surrogate key (`ROW_NUMBER`). |
| `customer_id` | integer | Business customer ID from CRM. |
| `customer_number` | string | Customer number/key (`cst_key`). |
| `first_name` | string | Customer first name. |
| `last_name` | string | Customer last name. |
| `country` | string | Country from ERP location mapping. |
| `marital_status` | string | Standardized marital status from CRM silver. |
| `gender` | string | Preferred CRM gender; fallback to ERP gender; default `n/a`. |
| `birthdate` | date | Birth date from ERP customer table. |
| `create_date` | date | Customer create date from CRM. |

## `gold.dim_products`
| Column | Type | Description |
|---|---|---|
| `product_key` | integer | Surrogate key (`ROW_NUMBER`). |
| `product_id` | integer | Product ID. |
| `product_number` | string | Product business key. |
| `product_name` | string | Product name. |
| `category_id` | string | Category ID. |
| `category` | string | Category name from ERP mapping. |
| `subcategory` | string | Subcategory name from ERP mapping. |
| `maintenance` | string | Maintenance flag from ERP mapping. |
| `cost` | numeric | Product cost. |
| `product_line` | string | Product line description. |
| `start_date` | date | Current version start date. |

**Filter logic:** only current records (`prd_end_dt IS NULL`).

## `gold.fact_sales`
| Column | Type | Description |
|---|---|---|
| `order_number` | string | Sales order number. |
| `product_key` | integer | FK to `gold.dim_products.product_key`. |
| `customer_key` | integer | FK to `gold.dim_customers.customer_key`. |
| `order_date` | date | Order date. |
| `shipping_date` | date | Shipping date. |
| `due_date` | date | Due date. |
| `sales_amount` | numeric | Sales amount. |
| `quanity` | integer | Quantity sold *(spelled `quanity` in script)*. |
| `price` | numeric | Unit price. |

---

## 6) Business Keys & Joins

## Primary business identifiers
- Customer business ID: `cst_id` (CRM), customer number: `cst_key`
- Product business key: transformed `prd_key` (Silver) / `product_number` (Gold)
- Sales order number: `sls_ord_num`

## Join patterns used
- `silver.crm_cust_info.cst_key = silver.erp_cust_az12.cid`
- `silver.crm_cust_info.cst_key = silver.erp_loc_a101.cid`
- `silver.crm_prd_info.cat_id = silver.erp_px_cat_g1v2.id`
- `silver.crm_sales_details.sls_prd_key = gold.dim_products.product_number`
- `silver.crm_sales_details.sls_cust_id = gold.dim_customers.customer_id`

---

## 7) Data Quality Rules Implemented

- Trim whitespace from customer names.
- Map marital status codes:
  - `S → Single`
  - `M → Married`
  - else `n/a`
- Map gender codes:
  - `F/FEMALE → Female`
  - `M/MALE → Male`
  - else `n/a`
- Remove `NAS` prefix from ERP customer IDs.
- Remove hyphens from ERP location customer IDs.
- Normalize country values (`DE`, `US`, `USA`, blanks).
- Replace null product cost with `0`.
- Correct invalid sales and price values using quantity/price consistency logic.
- Convert integer date fields (`YYYYMMDD`) to proper `DATE`, set invalid values to `NULL`.
- Deduplicate customers by latest `cst_create_date`.
- Build product effective end date using `LEAD(prd_start_dt) - 1`.

---

## 8) Known Naming/Script Notes (for future cleanup)

These are present in current scripts and may be standardized later:
- `cst_material_status` appears to be intended as `cst_marital_status`.
- `gold.fact_sales.quanity` appears to be intended as `quantity`.
- Some formatting issues in SQL scripts (extra spaces in object names) are cosmetic but should be reviewed.
- In `gold` DDL, ensure each `CREATE VIEW` statement is properly terminated (e.g., with `;`) in your runtime environment.

---

## 9) Suggested Ownership & Update Process

When schema changes:
1. Update DDL in `scripts/bronze`, `scripts/silver`, or `scripts/gold layer`.
2. Update this dictionary file in `docs/data-dictionary.md`.
3. Add a short changelog section in README (optional but recommended).

---

## 10) Version

- Catalog generated for commit based on repository snapshot (branch: `main`).
- File location recommendation: `docs/data-dictionary.md`.
