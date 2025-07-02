# Retail Sales Analysis SQL Project

## Project Overview

**Project Title**: Retail Sales Analysis  
**Level**: Beginner  
**Database**: `sql_project_p1`

This project demonstrates core SQL skills for data analysts by simulating a real-world retail sales scenario. It involves setting up a PostgreSQL database, cleaning and exploring the dataset, and answering common business questions using SQL queries. The analysis focuses on transaction-level sales data with attributes like time, customer demographics, and product categories.

---

## Objectives

1. **Database Setup** – Create a PostgreSQL database and table for storing retail sales data.
2. **Data Cleaning** – Detect and remove null or missing values to maintain data integrity.
3. **Exploratory Data Analysis (EDA)** – Explore basic metrics like total sales, unique customers, and product categories.
4. **Business Insights** – Answer 10 key business questions using SQL queries.

---

## 1. Database Setup

```sql
CREATE DATABASE sql_project_p1;

-- Use this DB in your session
-- \c sql_project_p1

DROP TABLE IF EXISTS retail_sales;

CREATE TABLE retail_sales (
    transactions_id INT PRIMARY KEY,
    sale_date DATE,
    sale_time TIME,
    customer_id INT,
    gender VARCHAR(15),
    age INT,
    category VARCHAR(15),
    quantity INT,
    price_per_unit FLOAT,
    cogs FLOAT,
    total_sale FLOAT
);
```

---

## 2. Data Cleaning & Exploration

```sql
-- Preview Data
SELECT * FROM retail_sales LIMIT 10;

-- Count total records
SELECT COUNT(*) FROM retail_sales;

-- Count unique customers
SELECT COUNT(DISTINCT customer_id) FROM retail_sales;

-- Explore categories
SELECT COUNT(DISTINCT category) FROM retail_sales;
SELECT DISTINCT category FROM retail_sales;

-- Check for nulls
SELECT * FROM retail_sales
WHERE transactions_id IS NULL OR sale_date IS NULL OR sale_time IS NULL OR
      customer_id IS NULL OR gender IS NULL OR category IS NULL OR
      quantity IS NULL OR cogs IS NULL OR total_sale IS NULL;

-- Remove rows with nulls
DELETE FROM retail_sales
WHERE transactions_id IS NULL OR sale_date IS NULL OR sale_time IS NULL OR
      customer_id IS NULL OR gender IS NULL OR category IS NULL OR
      quantity IS NULL OR cogs IS NULL OR total_sale IS NULL;
```

---

## 3. Business Questions & SQL Solutions

### Q1: Sales on '2022-11-05'
```sql
SELECT * FROM retail_sales
WHERE sale_date = '2022-11-05';
```

### Q2: Clothing transactions with quantity ≥ 4 in Nov 2022
```sql
SELECT * FROM retail_sales
WHERE category = 'Clothing'
  AND TO_CHAR(sale_date, 'YYYY-MM') = '2022-11'
  AND quantity >= 4;
```

### Q3: Total and count of sales by category
```sql
SELECT category, SUM(total_sale) AS net_sale, COUNT(*) AS total_orders
FROM retail_sales
GROUP BY category;
```

### Q4: Avg. age of customers buying 'Beauty' products
```sql
SELECT category, ROUND(AVG(age), 2) AS avg_age
FROM retail_sales
WHERE category = 'Beauty'
GROUP BY category;
```

### Q5: Transactions with total_sale > 1000
```sql
SELECT * FROM retail_sales
WHERE total_sale > 1000;
```

### Q6: Transaction count by gender and category
```sql
SELECT category, gender, COUNT(*) AS total_trans
FROM retail_sales
GROUP BY category, gender
ORDER BY category;
```

### Q7: Best-selling month (avg. sale) in each year
```sql
-- Option 1: Using RANK()
SELECT year, month, avg_sale
FROM (
  SELECT EXTRACT(YEAR FROM sale_date)::INT AS year,
         EXTRACT(MONTH FROM sale_date)::INT AS month,
         AVG(total_sale) AS avg_sale,
         RANK() OVER(PARTITION BY EXTRACT(YEAR FROM sale_date) ORDER BY AVG(total_sale) DESC) AS rank
  FROM retail_sales
  GROUP BY 1, 2
) t1
WHERE rank = 1;

-- Option 2: Using CTE and MAX()
WITH monthly_avg AS (
  SELECT EXTRACT(YEAR FROM sale_date)::INT AS year,
         EXTRACT(MONTH FROM sale_date)::INT AS month,
         AVG(total_sale) AS avg_sale
  FROM retail_sales
  GROUP BY 1, 2
)
SELECT *
FROM monthly_avg m
WHERE avg_sale = (
  SELECT MAX(avg_sale)
  FROM monthly_avg
  WHERE year = m.year
)
ORDER BY year;
```

### Q8: Top 5 customers by total sales
```sql
SELECT customer_id, SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY customer_id
ORDER BY total_sales DESC
LIMIT 5;
```

### Q9: Unique customers by category
```sql
SELECT category, COUNT(DISTINCT customer_id) AS unique_customers
FROM retail_sales
GROUP BY category;
```

### Q10: Orders by time of day (shift)
```sql
-- Option 1: Inline CASE
SELECT 
  CASE
    WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
    WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
    ELSE 'Evening'
  END AS shift,
  COUNT(*) AS total_orders
FROM retail_sales
GROUP BY shift;

-- Option 2: CTE
WITH hourly_sale AS (
  SELECT *,
         CASE
           WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
           WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
           ELSE 'Evening'
         END AS shift
  FROM retail_sales
)
SELECT shift, COUNT(*) AS total_orders
FROM hourly_sale
GROUP BY shift;
```

---

## Key Findings

- **Customer Patterns**: Customers span various demographics with sales across multiple product categories.
- **High-Value Transactions**: Multiple large transactions highlight premium segments.
- **Sales Trends**: Monthly and hourly trends highlight peak seasons and customer shopping behavior.
- **Top Buyers**: Certain customers consistently rank as top contributors to revenue.

---

## Conclusion

This PostgreSQL-based SQL project demonstrates foundational data analysis skills including database setup, data exploration, cleaning, and deriving insights using queries. It's a solid example for beginner portfolios, showcasing practical SQL usage in business contexts.

---

## Author – Abishek Karnan Rajesh

This project is part of my data analytics portfolio, completed during my MS in Business Analytics at UConn. Feel free to connect for feedback, collaboration, or project walkthroughs.

Thank you for reading!
