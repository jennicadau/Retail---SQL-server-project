**INTRODUCTION**  
The project explores the   

**TOOLS AND TECHNIQUES   ** 

**ABOUT THE PROJECT  ** 

-- QUESTION 1:
/* How can you write a SQL query to calculate the total sales of furniture products,
grouped by each quarter of the year, and order the results chronologically? */
WITH CTE as (
	SELECT DATEPART(qq, ORDER_DATE) Q, DATEPART(yy, ORDER_DATE) Y, PRODUCT_ID, SALES
	FROM orders
)

SELECT 
		'Q' + CAST(c.Q AS varchar) + '-' + CAST(c.Y AS varchar) [Quarter_Year],
		ROUND(SUM(SALES),2) [Total_Sales]
FROM CTE c
JOIN product p ON c.PRODUCT_ID = p.ID
WHERE p.NAME LIKE 'Furniture'
GROUP BY 'Q' + CAST(c.Q AS varchar) + '-' + CAST(c.Y AS varchar), c.Y, c.Q
ORDER BY c.Y, c.Q;

-- QUESTION 2:
/* How can you analyze the impact of different discount levels on sales performance across product categories, 
specifically looking at the number of orders and total profit generated for each discount classification?

Discount level condition:
No Discount = 0
0 < Low Discount < 0.2
0.2 < Medium Discount < 0.5
High Discount > 0.5 */
SELECT 
	p.CATEGORY [Category], 
	CASE WHEN o.DISCOUNT = 0 THEN 'No Discount'
		WHEN o.DISCOUNT > 0 AND o.DISCOUNT<0.2 THEN 'Low Discount'
		WHEN o.DISCOUNT > 0.5 THEN 'High Discount'
		ELSE 'Medium Discount' END Discount_Level,
	COUNT(o.ORDER_ID) Total_orders,
	ROUND(SUM(o.PROFIT),2) Total_Profit
FROM orders o
JOIN product p ON o.PRODUCT_ID = p.ID
GROUP BY p.CATEGORY,
	CASE WHEN o.DISCOUNT = 0 THEN 'No Discount'
		WHEN o.DISCOUNT > 0 AND o.DISCOUNT<0.2 THEN 'Low Discount'
		WHEN o.DISCOUNT > 0.5 THEN 'High Discount'
		ELSE 'Medium Discount' END
ORDER BY Category;

-- QUESTION 3:
/* How can you determine the top-performing product categories within each customer segment based on sales and profit, 
focusing specifically on those categories that rank within the top two for profitability? */
WITH CTE AS (
	SELECT 
		c.SEGMENT Segment, 
		p.CATEGORY Category,
		ROW_NUMBER() OVER (Partition by c.SEGMENT ORDER BY sum(o.SALES) DESC) Sales_rank,
		ROW_NUMBER() OVER (Partition by c.SEGMENT ORDER BY sum(o.PROFIT) DESC) Profit_rank
	FROM orders o
	JOIN Customer c ON o.CUSTOMER_ID = c.ID
	JOIN product p ON o.PRODUCT_ID = p.ID
	GROUP BY c.SEGMENT, p.CATEGORY) 
SELECT Segment, Category, Sales_rank, Profit_rank
FROM CTE
WHERE Profit_rank IN (1,2);

-- QUESTION 4
/*
How can you create a report that displays each employee's performance across different product categories, 
showing not only the total profit per category but also what percentage of 
their total profit each category represents, with the results ordered by the 
percentage in descending order for each employee?
*/
WITH CTE AS (
	SELECT ID_EMPLOYEE, ROUND(sum(PROFIT),2) e_profit
	FROM orders 
	GROUP BY ID_EMPLOYEE
	)

SELECT 
	o.ID_EMPLOYEE, p.CATEGORY,
	ROUND(SUM(o.PROFIT),2) Total_profit,
	ROUND(SUM(o.PROFIT)/c.e_profit*100,2) Profit_percent
FROM orders o
JOIN CTE c ON o.ID_EMPLOYEE = c.ID_EMPLOYEE
JOIN product p ON o.PRODUCT_ID = p.ID
GROUP BY o.ID_EMPLOYEE, p.CATEGORY,e_profit
ORDER BY o.ID_EMPLOYEE, Profit_percent DESC;


-- QUESTION 5:
/*
How can you develop a user-defined function in SQL Server 
to calculate the profitability ratio for each product category an employee has sold, 
and then apply this function to generate a report that 
ranks each employee's product categories by their profitability ratio?
*/
SELECT 
	o.ID_EMPLOYEE, 
	p.CATEGORY,
	ROUND(SUM(o.SALES),2) Total_Sales,
	ROUND(SUM(o.PROFIT),2) Total_Profit,
	ROUND(SUM(o.PROFIT)/SUM(o.SALES),2) Profit_ratio
FROM orders o
JOIN product p ON o.PRODUCT_ID = p.ID
GROUP BY o.ID_EMPLOYEE,p.CATEGORY
ORDER BY o.ID_EMPLOYEE, Profit_ratio DESC;


