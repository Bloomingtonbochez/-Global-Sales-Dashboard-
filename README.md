# Global Sales Dashboard
This project showcases an end-to-end data analytics pipeline using SQL Server for data cleaning, consolidation, and transformation, and Power BI for interactive dashboards. It analyzes sales data from 6 countries, handles missing values, creates key metrics, and delivers insights on performance, trends, payments, and profit vs. discounts.
i combined sales data from 6 countries (Canada, China, India, Nigeria, UK, US), handled missing values, created calculated metrics, and built a dashboard to track sales performance, payment methods, seasonal trends, and discount vs. profit relationships.


## 🗂️ Dataset Description

Source: Six country-specific sales tables (sales_Canada, sales_China, etc.)

Rows per table: ~1K–5K transactions

Key columns:

```
Transaction_ID, Date, Country, Product_ID, Product_Name, Category, Price_per_Unit, Quantity_Purchased, Cost_Price, Discount_Applied, Payment_Method, Customer_Age_Group, Customer_Gender, Store_Location, Sales_Rep
```
## 🛠️ Tools Used

Tool	Purpose

SQL Server	Data union, cleaning, feature engineering

Power BI	Dashboard creation, DAX measures, visualizations

GitHub	Version control & portfolio hosting

## 📁 Repository Structure
```

global-sales-dashboard/
│
├── data/
│   └── sample_raw_data.csv (optional)
│
├── sql/
│   ├── 01_create_master_table.sql
│   ├── 02_data_cleaning.sql
│   └── 03_add_calculated_columns.sql
│
├── powerbi/
│   └── global_sales_dashboard.pbix
│
├── images/
│   └── dashboard_screenshot.png
│
├── README.md
└── requirements.txt (if using Python)
```

#🔧 SQL Data Preparation (Full Script)
## 1️⃣ Combine All Country Tables into One Master Table

```
IF OBJECT_ID('dbo.General_Sales_table','U') IS NOT NULL
    DROP TABLE dbo.General_Sales_table;

SELECT
    Transaction_ID,
    Date,
    Country,
    Product_ID,
    Product_Name,
    Category,
    Price_per_Unit,
    Quantity_Purchased,
    Cost_Price,
    Discount_Applied,
    Payment_Method,
    Customer_Age_Group,
    Customer_Gender,
    Store_Location,
    Sales_Rep
INTO General_Sales_table
FROM (
    SELECT * FROM sales_Canada
    UNION ALL
    SELECT * FROM sales_China
    UNION ALL
    SELECT * FROM sales_India
    UNION ALL
    SELECT * FROM sales_Nigeria
    UNION ALL
    SELECT * FROM sales_UK
    UNION ALL
    SELECT * FROM sales_US
) AS Combined;

```
## 2️⃣ Check for NULL Values
```
SELECT *
FROM General_Sales_table
WHERE Transaction_ID IS NULL 
   OR Country IS NULL
   OR Product_ID IS NULL
   OR Quantity_Purchased IS NULL
   OR Price_per_Unit IS NULL;
```

## 3️⃣ Impute Missing Price_per_Unit with Global Average
```
UPDATE General_Sales_table
SET Price_per_Unit = (
    SELECT AVG(Price_per_Unit)
    FROM General_Sales_table
    WHERE Price_per_Unit IS NULL
)
WHERE Transaction_ID = '001898f7-b696-4356-91dc-8f2b73d09c63';
```
## 4️⃣ Create Total_amount Column (After Discount)
```
ALTER TABLE General_Sales_table
ADD Total_amount NUMERIC(10,2);

UPDATE General_Sales_table
SET Total_amount = (Price_per_Unit * Quantity_Purchased) - Discount_Applied;
```
## 5️⃣ Create Profit Column
```
ALTER TABLE General_Sales_table
ADD Profit NUMERIC(10,2);

-- Profit = Revenue after discount - Total cost
UPDATE General_Sales_table
SET Profit = Total_amount - (Cost_Price * Quantity_Purchased);
```
## 6️⃣ Final Table for Power BI
```
SELECT 
    Transaction_ID,
    Date,
    YEAR(Date) AS Year,
    MONTH(Date) AS Month,
    DATENAME(MONTH, Date) AS Month_Name,
    DATENAME(WEEKDAY, Date) AS Day_of_Week,
    Country,
    Category,
    Payment_Method,
    Total_amount,
    Profit,
    Discount_Applied,
    Customer_Age_Group,
    Customer_Gender
FROM General_Sales_table;
```
## 📈 Power BI Dashboard – Key Features

 ```https://./Screenshot%25202026-04-24%2520234520.png```

#💡 Dashboard Business Recommendation
### 1. Payment Method
Launch mobile-exclusive 10% discount → Grow from 34% to 40% share

Keep cash for older demographics

### 2. Seasonal Strategy
Increase inventory 40% before January → Capture $2.6M Q1 peak

Run "Summer Sale" in May-June → Lift low months from $0.24M to $0.35M

### 3. Discount Optimization
Cap discounts at $2,500 in India & Nigeria → Protect profit margins

US/UK/Canada can keep higher discounts (they convert well)

### 4. Category Focus
Prioritize Home & Kitchen + Beauty (combined ~$1.3M sales)

Reduce Sports/Toys inventory (under $0.10M each)

### 5. Country Actions
Country	Action
US/UK	Increase discount budget
India/Nigeria	Reduce discounts 50%, add mobile payments
Canada	Launch loyalty program












