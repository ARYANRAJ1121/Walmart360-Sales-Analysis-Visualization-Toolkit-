# Walmart360: Advanced SQL Analysis Portfolio
![SQL](https://img.shields.io/badge/SQL-Advanced-blue)
![PowerBI](https://img.shields.io/badge/Power%20BI-Coming_Soon-yellow)
![Project-Status](https://img.shields.io/badge/Status-Active-green)

This repository demonstrates advanced SQL analytics and interview-ready queries using the `walmart_sales` dataset. The focus is on business-oriented SQL problems commonly asked in data analyst interviews.

> ✅ Visualizations using **Pandas, Seaborn, Matplotlib**, and **Power BI** are in progress and will be uploaded as development completes.

---

## 📘 Table of Contents

- [Project Overview](#project-overview)
- [Dataset Schema](#dataset-schema)
- [SQL Interview Questions & Solutions](#sql-interview-questions--solutions)
- [In-Progress: Visualization Toolkit](#in-progress-visualization-toolkit)
- [Planned Enhancements](#planned-enhancements)

---

## 📊 Project Overview

This project, **Walmart360**, is built entirely using SQL and structured around the Walmart sales forecasting dataset. It includes feature-rich SQL questions tackling fundamentals as well as advanced analytics with real-case business scenarios.

---

## 🧾 Dataset Schema

The structure of the table used:

~~~sql
CREATE TABLE walmart_sales (
    Store INT,
    Dept INT,
    Date DATE,
    Weekly_Sales FLOAT,
    IsHoliday BOOLEAN,
    Week INT,
    Year INT,
    Temperature FLOAT,
    Fuel_Price FLOAT,
    MarkDown1 FLOAT,
    MarkDown2 FLOAT,
    MarkDown3 FLOAT,
    MarkDown4 FLOAT,
    MarkDown5 FLOAT,
    CPI FLOAT,
    Unemployment FLOAT
);
~~~

---

## 🧠 SQL Interview Questions & Solutions

### ✅ Question 1: Basic Data Retrieval  
**Retrieve all stores with weekly sales greater than $50,000.**

~~~sql
SELECT Store
FROM walmart_sales
WHERE Weekly_Sales > 50000
ORDER BY Store DESC;
~~~

---

### ✅ Question 2: Aggregate Functions  
**Total weekly sales and average temperature per store.**

~~~sql
SELECT
  Store,
  SUM(Weekly_Sales) AS Total_Weekly_Sales,
  AVG(Temperature) AS Avg_Temperature
FROM walmart_sales
GROUP BY Store;
~~~

---

### ✅ Question 3: Date Filtering  
**Filter records from year 2011 where `IsHoliday` is TRUE.**

~~~sql
SELECT *
FROM walmart_sales
WHERE Year = 2011 AND IsHoliday IS TRUE;
~~~

---

### ✅ Question 4: Sorting and Limiting  
**Top 10 sales records across stores.**

~~~sql
SELECT Store, Weekly_Sales
FROM walmart_sales
ORDER BY Weekly_Sales DESC
LIMIT 10;
~~~

---

### ✅ Question 5: NULL Value Handling  
**Count NULLs in `MarkDown1`.**

~~~sql
SELECT COUNT(*) AS Null_MarkDown1_Count
FROM walmart_sales
WHERE MarkDown1 IS NULL;
~~~

---

### ✅ Question 6: String and LIKE  
**Find unemployment values that start with “8.”**

~~~sql
SELECT *
FROM walmart_sales
WHERE CAST(Unemployment AS TEXT) LIKE '8.%';
~~~

---

### ✅ Question 7: CTE Query  
**Stores with above-average weekly sales.**

~~~sql
WITH avg_sales AS (
  SELECT AVG(Weekly_Sales) AS overall_avg FROM walmart_sales
)
SELECT DISTINCT ws.Store
FROM walmart_sales AS ws
JOIN avg_sales AS a ON ws.Weekly_Sales > a.overall_avg
ORDER BY ws.Store;
~~~

---

### ✅ Question 8: CASE Statements  
**Categorize sales as Low, Medium, High.**

~~~sql
SELECT
  Store,
  Dept,
  Date,
  Weekly_Sales,
  CASE
    WHEN Weekly_Sales < 20000 THEN 'Low'
    WHEN Weekly_Sales BETWEEN 20000 AND 50000 THEN 'Medium'
    ELSE 'High'
  END AS Sales_Category
FROM walmart_sales
ORDER BY Store, Date;
~~~

---

### ✅ Question 9: Multi-Filter Join  
**Get records where `Temperature > 70` and `Fuel_Price > 3`.**

~~~sql
SELECT
  t1.Store,
  t1.Dept,
  t1.Date,
  t1.Weekly_Sales,
  t1.Temperature,
  t2.Fuel_Price
FROM walmart_sales AS t1
JOIN walmart_sales AS t2
  ON t1.Store = t2.Store AND t1.Dept = t2.Dept AND t1.Date = t2.Date
WHERE t1.Temperature > 70 AND t2.Fuel_Price > 3.00
ORDER BY t1.Date, t1.Store;
~~~

---

### ✅ Question 10: Month-over-Month Sales Growth  
**Calculate MoM growth using date functions.**

~~~sql
WITH monthlysales AS (
  SELECT store, Year, EXTRACT(MONTH FROM Date) AS Month,
         SUM(Weekly_Sales) AS Total_Sales
  FROM walmart_sales
  GROUP BY store, Year, EXTRACT(MONTH FROM Date)
),
saleswithlag AS (
  SELECT store, Year, Month, Total_Sales,
         LAG(Total_Sales) OVER (PARTITION BY store ORDER BY Year, Month) AS Prev_Month_Sales
  FROM monthlysales
)
SELECT
  store, Year, Month, Total_Sales, Prev_Month_Sales,
  (Total_Sales - Prev_Month_Sales) * 100.0 / NULLIF(Prev_Month_Sales, 0) AS MoM_Growth_Percentage
FROM saleswithlag
ORDER BY store, Year, Month;
~~~

---

### ✅ Question 11: GROUP BY with HAVING  
**Departments with >$100k sales & >10 weeks.**

~~~sql
SELECT Dept, SUM(Weekly_Sales) AS Total_Sales, COUNT(*) AS Week_Count
FROM walmart_sales
GROUP BY Dept
HAVING SUM(Weekly_Sales) > 100000 AND COUNT(*) > 10
ORDER BY Total_Sales DESC;
~~~

---

### ✅ Question 12: ROW_NUMBER  
**Rank top 3 stores per department.**

~~~sql
WITH DeptSales AS (
  SELECT Dept, Store, SUM(Weekly_Sales) AS Total_Sales
  FROM walmart_sales
  GROUP BY Dept, Store
),
RankedSales AS (
  SELECT Dept, Store, Total_Sales,
         ROW_NUMBER() OVER (PARTITION BY Dept ORDER BY Total_Sales DESC) AS Sales_Rank
  FROM DeptSales
)
SELECT Dept, Store, Total_Sales, Sales_Rank
FROM RankedSales
WHERE Sales_Rank <= 3
ORDER BY Dept, Sales_Rank;
~~~

---

### ✅ Question 13: Correlation  
**Correlation between sales and unemployment.**

~~~sql
SELECT ROUND(corr(Weekly_Sales, Unemployment)::numeric, 4) AS sales_unemployment_correlation
FROM walmart_sales;
~~~

---

### ✅ Question 14: Running Total  
**Running total of weekly sales per store.**

~~~sql
WITH salesbydate AS (
  SELECT Store, Date, Weekly_Sales FROM walmart_sales
),
runningtotal AS (
  SELECT Store, Date, Weekly_Sales,
         SUM(Weekly_Sales) OVER(PARTITION BY Store ORDER BY Date ROWS UNBOUNDED PRECEDING) AS Running_Total
  FROM salesbydate
)
SELECT Store, Date, Weekly_Sales, ROUND(Running_Total::numeric, 2) AS Running_Total
FROM runningtotal
ORDER BY Store, Date;
~~~

---

### ✅ Question 15: Week-over-Week Change (LAG)  
~~~sql
WITH sales_by_store_week AS (
  SELECT Store, Date, SUM(Weekly_Sales) AS Weekly_Sales
  FROM walmart_sales
  GROUP BY Store, Date
),
with_lag AS (
  SELECT Store, Date, Weekly_Sales,
         LAG(Weekly_Sales) OVER (PARTITION BY Store ORDER BY Date) AS Prev_Week_Sales
  FROM sales_by_store_week
)
SELECT Store, Date, Weekly_Sales, Prev_Week_Sales,
  ROUND((Weekly_Sales - Prev_Week_Sales)::numeric, 2) AS Difference,
  CASE
    WHEN Prev_Week_Sales = 0 OR Prev_Week_Sales IS NULL THEN NULL
    ELSE ROUND(((Weekly_Sales - Prev_Week_Sales) * 100.0 / Prev_Week_Sales)::numeric, 2)
  END AS WoW_Change_Percent
FROM with_lag
ORDER BY Store, Date;
~~~

---

### ✅ Question 16: Percentiles using PERCENTILE_CONT  
~~~sql
WITH t1 AS (
  SELECT
    PERCENTILE_CONT(0.1) WITHIN GROUP (ORDER BY Weekly_Sales) AS p10,
    PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY Weekly_Sales) AS p90
  FROM walmart_sales
)
SELECT ws.Store, ws.Date, ws.Weekly_Sales,
  CASE
    WHEN ws.Weekly_Sales >= p.p90 THEN 'Top 10%'
    WHEN ws.Weekly_Sales <= p.p10 THEN 'Bottom 10%'
    ELSE 'Middle 80%'
  END AS Sales_Band
FROM walmart_sales ws
CROSS JOIN t1 p
ORDER BY ws.Weekly_Sales DESC;
~~~

---

## 📈 In-Progress: Visualization Toolkit

> 📌 Coming soon:  
- 📊 Seaborn & Matplotlib visual plots (20+ graph types)  
- 🔍 Exploratory Data Analysis (EDA) Utilities  
- 📈 Power BI Dashboards  

## 🚧 Planned Enhancements

- ✅ SQL Queries (Complete)  
- 🟡 Visual Notebooks in Seaborn & Matplotlib (In Progress)  
- 🔵 Power BI Dashboards for Executive Summaries (Coming Soon)  

---

## 🚀 How to Use This Repository

1. Clone the project or copy individual queries to practice.
2. Watch the **visualization** branch for future updates.
3. Explore and modify each use case for real-world data challenges.

💡 **Star** this repo if it helped your interview prep or portfolio!

---
