# 🛒 E-Commerce Analysis — Olist Dataset

## 📌 Project Overview
This is an end-to-end data analysis project based on the **Olist Brazilian E-Commerce Dataset**. The project covers the complete data analyst workflow — from raw data to business insights — using Python, SQL Server, and Power BI.

The goal was to analyze 100,000+ orders placed on the Olist platform between 2016 and 2018 and uncover actionable business insights around sales, customer behavior, delivery performance, and seller efficiency.

---

## 🎯 Business Questions Answered
- What is the overall revenue and growth trend?
- Which states and cities drive the most orders?
- Are deliveries being made on time?
- What percentage of customers are repeat buyers?
- Which sellers are most and least efficient?
- Where is the business losing money through freight costs?

---

## 🗂️ Dataset
**Source:** [Olist Brazilian E-Commerce Dataset — Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

### Files Used:
| File | Description | Rows |
|---|---|---|
| olist_customers_dataset.csv | Customer information | 99,441 |
| olist_orders_dataset.csv | Order details and timestamps | 99,441 |
| olist_order_items_dataset.csv | Items within each order | 112,650 |
| olist_order_payments_dataset.csv | Payment information | 103,886 |
| olist_order_reviews_dataset.csv | Customer reviews | 99,224 |
| olist_products_dataset.csv | Product details | 32,951 |
| olist_sellers_dataset.csv | Seller information | 3,095 |
| olist_geolocation_dataset.csv | Zip code coordinates | 1,000,163 |
| product_category_name_translation.csv | Category translations | 71 |

---

## 🛠️ Tools & Technologies
| Tool | Purpose |
|---|---|
| Python (Pandas) | Data loading, cleaning, EDA |
| Jupyter Notebook | Exploratory analysis environment |
| SQL Server (SSMS) | Business analysis queries |
| Power BI | Dashboard and visualization |
| GitHub | Project documentation |

---

## 📁 Project Structure
```
Olist_Dataset/
│
├── data/
│   ├── raw/                        # Original CSV files
│   └── cleaned/                    # Cleaned CSV files
│       ├── customers_cleaned.csv
│       ├── orders_cleaned.csv
│       └── order_items_cleaned.csv
│
├── notebooks/
│   └── Data_Cleaning.ipynb         # Python EDA and cleaning
│
├── sql/
│   ├── 01_sales_revenue.sql        # Sales analysis queries
│   ├── 02_customer_geography.sql   # Customer analysis queries
│   ├── 03_delivery_performance.sql # Delivery analysis queries
│   ├── 04_order_behaviour.sql      # Order behavior queries
│   └── 05_freight_shipping.sql     # Freight analysis queries
│
├── powerbi/
│   └── Olist_Report.pbix           # Power BI dashboard file
│
└── README.md
```

---

## 🔄 Project Workflow

### Step 1 — Data Loading (Python)
- Loaded 3 core CSV files using Pandas
- Verified shapes: Customers (99,441 x 5), Orders (99,441 x 8), Order Items (112,650 x 7)

### Step 2 — Exploratory Data Analysis (Python)
- Checked data types, missing values, and duplicates
- Found missing values in 3 date columns in Orders table
- Zero duplicates across all 3 datasets

### Step 3 — Data Cleaning (Python + SQL)
- Converted 5 date columns from string to datetime format
- Filled missing date values using average time difference method
- Removed orders with no delivery date (kept 96,476 delivered orders)

### Step 4 — Business Analysis (SQL Server)
- Wrote 20+ analytical SQL queries across 5 business areas
- Used CTEs, Window Functions, JOINs, and CASE WHEN statements

### Step 5 — Visualization (Power BI)
- Built 3-page interactive dashboard
- Created DAX measures for KPIs and ratings
- Connected all 3 tables through data model relationships

---

## 📊 Power BI Dashboard

### Page 1 — Executive Summary
| Visual | Insight |
|---|---|
| KPI Cards | R$13.59M revenue, 96,476 orders, 96,096 customers, 12.5 days avg delivery |
| Revenue Trend | 10x growth from R$112K to R$988K in 2017 |
| On Time vs Late | 91.89% on time, 8.11% late |
| Top 5 States | SP dominates with R$5.1M (3x RJ) |

### Page 2 — Customer Analysis
| Visual | Insight |
|---|---|
| Brazil State Map | Customer concentration in Southeast Brazil |
| Top 10 Cities | Sao Paulo = 15K orders (2x Rio de Janeiro) |
| Buyer Type | 96.88% one time buyers — serious retention issue |
| Peak Hours | Afternoon hours drive most orders |

### Page 3 — Seller & Delivery Deep Dive
| Visual | Insight |
|---|---|
| Delivery Days by State | Northern states 25+ days vs Southern 12 days |
| Late Deliveries Trend | Spikes during high volume months |
| Top 10 Sellers | Heavy revenue concentration in top sellers |
| Seller Ratings | Distribution of Excellent/Good/Average/Poor sellers |

---

## 🔍 Key SQL Queries

### Average Order Value:
```sql
WITH AVG_Order_Value AS (
    SELECT order_id, SUM(price) AS revenue
    FROM order_items
    GROUP BY order_id
)
SELECT AVG(revenue) AS avg_order_value
FROM AVG_Order_Value
```

### Month Over Month Revenue Growth:
```sql
WITH Monthly_Revenue AS (
    SELECT
        YEAR(o.order_purchase_timestamp) AS year,
        MONTH(o.order_purchase_timestamp) AS month,
        SUM(price) AS revenue
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    GROUP BY YEAR(o.order_purchase_timestamp), MONTH(o.order_purchase_timestamp)
)
SELECT
    year, month, revenue,
    LAG(revenue, 1) OVER (ORDER BY year, month) AS prev_revenue,
    ROUND(((revenue - LAG(revenue,1) OVER (ORDER BY year,month)) /
    LAG(revenue,1) OVER (ORDER BY year,month)) * 100, 2) AS growth_percentage
FROM Monthly_Revenue
ORDER BY year, month
```

### Late vs On Time Delivery:
```sql
SELECT
    CASE WHEN order_delivered_customer_date > order_estimated_delivery_date
         THEN 'Late' ELSE 'On Time'
    END AS delivery_status,
    COUNT(order_id) AS total_orders
FROM orders
WHERE order_delivered_customer_date IS NOT NULL
GROUP BY CASE WHEN order_delivered_customer_date > order_estimated_delivery_date
              THEN 'Late' ELSE 'On Time' END
```

### Seller Rating by Freight Efficiency:
```sql
SELECT
    seller_id,
    COUNT(order_id) AS total_orders,
    ROUND(AVG(price), 2) AS avg_price,
    ROUND(AVG(freight_value), 2) AS avg_freight,
    ROUND(AVG(freight_value/price)*100, 2) AS freight_percentage,
    CASE WHEN AVG(freight_value/price)*100 < 10 THEN 'Excellent'
         WHEN AVG(freight_value/price)*100 < 25 THEN 'Good'
         WHEN AVG(freight_value/price)*100 < 50 THEN 'Average'
         ELSE 'Poor'
    END AS seller_rating
FROM order_items
WHERE price != 0 AND price IS NOT NULL
GROUP BY seller_id
ORDER BY freight_percentage ASC
```

---

## 💡 Key Business Insights

### 1. Revenue & Growth
- Total revenue of **R$13.59M** across 2 years
- Business grew **10x** from R$112K (Jan 2017) to R$988K (Nov 2017)
- Revenue stabilized at **R$800K-R$978K** throughout 2018

### 2. Geography
- **Sao Paulo** accounts for **R$5.1M** — nearly 3x the second state
- Top 3 cities (SP, RJ, BH) are all in **Southeast Brazil**
- Northern states have very low customer penetration — untapped market

### 3. Customer Retention
- **96.88% of customers never return** — critical retention problem
- Only **3.12% repeat buyers** — loyalty program urgently needed
- High customer acquisition cost with low lifetime value

### 4. Delivery Performance
- **8.11% of orders delivered late** — 1 in 12 customers affected
- Northern states average **25+ days** delivery vs 12 days in South
- Gap of **13+ days** between best and worst performing states

### 5. Seller Efficiency
- Heavy revenue concentration in **top 10 sellers**
- Poor rated sellers have freight costs **50%+ of item price**
- Excellent sellers keep freight below **10% of item price**

---

## 📋 Recommendations

| Problem | Recommendation |
|---|---|
| 96.88% one time buyers | Launch loyalty/rewards program |
| Northern states slow delivery | Partner with local logistics providers |
| SP city dominance | Expand marketing to MG, RS, PR states |
| Poor seller freight costs | Set maximum freight % policy for sellers |
| Late delivery spikes | Add buffer capacity during peak months |

---

## 🚀 How to Run This Project

### Python:
```bash
pip install pandas openpyxl jupyter
cd "D:\DA Projects\Olist_Dataset"
jupyter notebook
```
Open `Data_Cleaning.ipynb` and run all cells

### SQL:
1. Open SQL Server Management Studio
2. Create database: `CREATE DATABASE OlistDB`
3. Import cleaned CSV files
4. Run queries from `/sql/` folder in order

### Power BI:
1. Open `Olist_Report.pbix`
2. Update data source path if needed
3. Click Refresh

---

## 👤 Author
**Prit Patel**
- Aspiring Data Analyst
- Tools: Python | SQL | Power BI

---

## 📄 License
This project uses publicly available data from Kaggle under the CC BY-NC-SA 4.0 license.
