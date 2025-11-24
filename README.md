Can't create, edit or upload … If your storage is full for two or more years, your files may be deleted from Drive and Photos.
# IKEA Retail Sales SQL Project

![Project Banner Placeholder](https://github.com/Jyothi-Raju122/Ikea/blob/main/Ikea-logo.png)

Welcome to the **IKEA Retail Sales SQL Project**! This project leverages a detailed dataset of millions of sales records, product inventory, and store information across IKEA's global operations. The analysis focuses on uncovering sales trends, product performance, and inventory management insights to assist in data-driven decision-making.

---

## Table of Contents
- [Introduction](#introduction)
- [Project Structure](#project-structure)
- [Database Schema](#database-schema)
- [Business Problems](#business-problems)
- [SQL Queries & Analysis](#sql-queries--analysis)
- [Getting Started](#getting-started)
- [Questions & Feedback](#questions--feedback)
- [Contact Me](#contact-me)
- [ERD (Entity-Relationship Diagram)](#erd-entity-relationship-diagram)

---

## Introduction

The IKEA Retail Sales SQL Project demonstrates the use of SQL to analyze retail data, including **sales records**, **store performance**, **product trends**, and **inventory status**. Using a robust schema, this project answers critical business questions and provides actionable insights to optimize IKEA's operational efficiency and profitability.

---

## Project Structure

1. **SQL Scripts**: Contains SQL queries to create the database schema, populate tables, and perform analyses.
2. **Dataset**: Includes sales data, product information, store details, and inventory records.
3. **Analysis**: SQL queries solve key business problems, leveraging advanced SQL techniques like joins, aggregations, and subqueries.

---

## Database Schema

### 1. **Products Table**
- **product_id**: Unique identifier for each product (Primary Key).
- **product_name**: Name of the product.
- **category**: Category to which the product belongs.
- **subcategory**: Subcategory of the product.
- **unit_price**: Price per unit of the product.

### 2. **Stores Table**
- **store_id**: Unique identifier for each store (Primary Key).
- **store_name**: Name of the store.
- **city**: City where the store is located.
- **country**: Country where the store operates.

### 3. **Sales Table**
- **order_id**: Unique identifier for each sales order (Primary Key).
- **order_date**: Date when the order was placed.
- **product_id**: Foreign key referencing the `products` table.
- **qty**: Quantity of the product sold.
- **discount_percentage**: Discount applied to the order.
- **unit_price**: Price per unit of the product at the time of sale.
- **store_id**: Foreign key referencing the `stores` table.

### 4. **Inventory Table**
- **inventory_id**: Unique identifier for each inventory record (Primary Key).
- **product_id**: Foreign key referencing the `products` table.
- **current_stock**: Current stock level of the product.
- **reorder_level**: Minimum stock level to trigger a reorder.

---

## Business Problems

This project tackles the following business problems:

### Easy-Level Queries
1. Identify the top 5 best-selling products.
```SQL
SELECT
	s.product_id,
	p.product_name,
	COUNT(order_id)
FROM
	products p
	JOIN
	sales s
	ON s.product_id = p.product_id
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 5

```

2. List all products that are low in stock.
```sql
SELECT
	i.product_id,
	p.product_name,
	i.current_stock,
	i.reorder_level
FROM
	inventory i
	JOIN
	products p
	ON p.product_id = i.product_id
WHERE
	i.current_stock < i.reorder_level
```



3. Calculate total sales revenue for each store.
```sql

SELECT
	s.store_id,
	st.store_name,
	ROUND(SUM(s.net_sales)::NUMERIC,2)
FROM
	sales s  
	JOIN stores st  
	ON st.store_id = s.store_id
GROUP BY 1,2

```

4. Find the top 3 stores with the highest sales in a specific country.
```sql

SELECT 
    s.store_name, 
    SUM(sales.qty * sales.unit_price) AS total_revenue
FROM 
    sales
JOIN 
    stores s ON sales.store_id = s.store_id
WHERE 
    s.country = 'USA'
GROUP BY 
    s.store_name
ORDER BY 
    total_revenue DESC;
```


5. Retrieve sales data for the last 6 months.
```sql
SELECT
	*
FROM
	sales
WHERE
	order_date BETWEEN '2023-06-01' AND '2023-12-31'
ORDER BY order_date ASC

```

### Medium to Hard-Level Queries


1.Identify the top three products with the highest sales quantity in each country.
```SQL

WITH T1
AS
(
SELECT
	s.product_id,
	p.product_name,
	st.country,
	DENSE_RANK() OVER(PARTITION BY st.country ORDER BY SUM(s.qty) DESC) AS Rank,
	SUM(s.qty)
FROM
	sales s
	JOIN
	products p 
	ON p.product_id = s.product_id
	JOIN
	stores st
	ON s.store_id = st.store_id
GROUP BY 1,2,3
)
SELECT 
		*
FROM 
	T1
WHERE
	Rank <=3

```
2.Find stores where the total sales revenue is higher than the average revenue across all stores.
```SQL
SELECT
	store_id,
	store_name,
	sum(net_sales) AS store_sales
FROM
	global_sales
GROUP BY 1,2
HAVING sum(net_sales) > 
					    (SELECT
	                           sum(net_sales)/
				                              (SELECT COUNT(DISTINCT store_id) FROM global_sales) ----NESTED SUBQUERY.This gives count of stores
                         FROM
	                         global_sales)
```
3.Display the reorder status for each product in inventory as "Low Stock" if current stock is below the reorder level, otherwise "Sufficient Stock."
```SQL
SELECT *
FROM
	(  --Subquery starts from here
	SELECT
	i.inventory_id,
	i.current_stock,
	i.reorder_level,
	p.product_name,
	p.category,
	CASE
		WHEN current_stock < reorder_level THEN 'under_stock'
		ELSE 'sufficient_stock'
	END AS stock_status
FROM
	inventory AS i
LEFT JOIN
	products AS p
	ON p.product_id = i.product_id
	)
	AS t1 --Temporary table within ()
WHERE stock_status = 'under_stock'
```
4.Yearly revenue growth ratio.
```SQL
WITH Yearly_Revenue
AS
(
SELECT
	store_name,
	EXTRACT(YEAR FROM order_date) AS Year,
	-- LAG(SUM(net_sales),2) OVER(PARTITION BY store_name ORDER BY EXTRACT(YEAR FROM order_date)) AS prev2_rev, --LAG to extract previous 2 rows
	--SUM(net_sales)
	ROUND(SUM(net_sales)::"numeric",2) AS Revenue
FROM
	global_sales
GROUP BY 1,2
ORDER BY 1,2
),
Revenue_2t2
AS
(
SELECT
	store_name,
	Year,
	revenue AS Current_year_revenue,
	LAG(Revenue) OVER(PARTITION BY store_name ORDER BY Year) AS Previous_year_revenue
FROM
	Yearly_Revenue
)
SELECT
	store_name,
	Year,
	current_year_revenue,
	previous_year_revenue,
	(current_year_revenue-previous_year_revenue)::"numeric"/previous_year_revenue::"numeric" * 100
FROM
	Revenue_2t2;
```

5.Retrieve the total revenue and discount given on each product category per store.
```SQL
SELECT
category,
ROUND(sum(net_sales)::Numeric,2),
ROUND(avg(discount_percentage)::Numeric,2),
--discount_percentage,
RANK() OVER(PARTITION BY category ORDER BY ROUND(sum(net_sales)::Numeric,2) DESC ) AS Rank
FROM
	global_sales
GROUP BY 1
ORDER BY ROUND(sum(net_sales)::Numeric,2) DESC
```

---

## SQL Queries & Analysis

All SQL queries developed for this project are available in the `queries.sql` file. The queries demonstrate advanced SQL skills, including:

- Aggregations with `GROUP BY`.
- Filtering data using `WHERE` and `HAVING`.
- Joining multiple tables to uncover insights.
- Using subqueries and window functions for complex analyses.

---

## Getting Started

### Prerequisites
- PostgreSQL (or any SQL-compatible database).
- Basic knowledge of SQL.

### Steps to Run
1. **Clone the Repository**:
   ```bash
   git clone https://github.com/yourusername/ikea-sales-sql-project.git
   ```
2. **Set Up the Database**:
   - Run `schema.sql` to create the database schema.
   - Populate tables with sample data using `data.sql`.

3. **Execute Queries**:
   - Open `queries.sql` and execute the queries for analysis.

---

## Questions & Feedback

Feel free to reach out with questions or suggestions. Here's an example query for reference:



---

## Contact Me

📧 [Email] Jyothikrishnaraju122@gmail.com  
📞 Phone: 647-965-5153

---

## ERD (Entity-Relationship Diagram)

Here’s the ERD for the IKEA Retail Sales SQL Project:

![ERD Placeholder](https://github.com/najirh/sql-b01-ikea/blob/main/IKEA.png)

---
