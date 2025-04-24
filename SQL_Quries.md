# SQL Analysis: Fetch Dataset Insights

This file contains annotated SQL queries used to analyze the Fetch Takehome dataset. All queries were developed and tested using ****SQL Server Management Studio (SSMS).****

---

## 1. Top 5 Brands by Receipts Scanned (Age ≥ 21)
This query finds the top 5 brands based on the number of distinct receipts scanned by users who are 21 or older.

```
SELECT TOP 5
    p.BRAND,
	-- Calculating the unique receipts 
    COUNT(DISTINCT t.RECEIPT_ID) AS receipts_scanned
FROM dbo.transactions_cleaned t
JOIN dbo.users_cleaned u
    ON t.USER_ID = u.ID
JOIN dbo.products_cleaned p
    ON t.BARCODE = p.BARCODE
	-- Filtering out the users who are 21 and over. 
WHERE DATEDIFF(YEAR, u.BIRTH_DATE, GETDATE()) >= 21
GROUP BY p.BRAND
ORDER BY Receipts_Scanned DESC.

```
## 2. Top 5 Brands by Total Sales (Account Duration≥ 6 Months)
This query calculates and ranks brands by total sales, filtering for users who are active for at least 6 months
```
-- Calculating total sales per brand and ranking them
WITH BrandSales AS (
    SELECT
        p.BRAND            AS Brand,        -- Product brand
   -- Sum of all sales for that brand
		SUM(t.FINAL_SALE)  AS Total_Sales,  -- Total sales per brand
	-- Ranking of brands by sales 
        ROW_NUMBER()
          OVER (ORDER BY SUM(t.FINAL_SALE) DESC) AS ranks
    FROM dbo.transactions_cleaned AS t
    INNER JOIN dbo.users_cleaned    AS u
        ON t.USER_ID = u.ID
    INNER JOIN dbo.products_cleaned AS p
        ON t.BARCODE = p.BARCODE
    WHERE
    -- Filter out rows where sales value is null and ensure the user has had an account for≥ 6 months
		t.FINAL_SALE IS NOT NULL
        AND DATEDIFF(MONTH, u.CREATED_DATE, t.PURCHASE_DATE) >= 6
    GROUP BY
        p.BRAND
)
SELECT
    Brand,
    Total_Sales
FROM BrandSales
WHERE ranks <= 5
ORDER BY Total_Sales DESC;

```

## 3. Sales Percentage of Health & Wellness Products by Generation

This Query Segments users by generation and calculates the share  Health & Wellness products among their total sales
```
WITH Generations AS (
    SELECT
        u.ID AS USER_ID,
        -- Creating the categories by age groups
		CASE
            WHEN DATEDIFF(YEAR, u.BIRTH_DATE, GETDATE()) < 25 THEN 'Gen_Z'
            WHEN DATEDIFF(YEAR, u.BIRTH_DATE, GETDATE()) BETWEEN 25 AND 39 THEN 'Millennials'
            WHEN DATEDIFF(YEAR, u.BIRTH_DATE, GETDATE()) BETWEEN 40 AND 54 THEN 'Gen_X'
            WHEN DATEDIFF(YEAR, u.BIRTH_DATE, GETDATE()) >= 55 THEN 'Boomers'
			ELSE 'Unknown'  
        END AS Generation
    FROM dbo.users_cleaned AS u
),
Category_Sales AS (
    SELECT
        g.Generation,
		-- Sum of sales for all categories among generations
        SUM(t.FINAL_SALE)   AS Total_Sales,   
        -- Sum of Health_Wellness category sales among generations
		SUM(
            CASE WHEN p.CATEGORY_1 = 'Health & Wellness' THEN t.FINAL_SALE ELSE 0 END
        ) AS Health_Wellness_sales    
    FROM dbo.transactions_cleaned AS t
    INNER JOIN Generations        AS g
        ON t.USER_ID = g.USER_ID           
    INNER JOIN dbo.products_cleaned p
        ON t.BARCODE = p.BARCODE            
    WHERE
        -- Excluding sale data which is missing
		t.FINAL_SALE IS NOT NULL            
    GROUP BY
        g.Generation                       
)
SELECT
    Generation,                          
     -- Calculating the Percentage of sales in the Health_Wellness category
	ROUND(
		CAST(Health_Wellness_sales   AS DECIMAL(18,2)) * 100.0 / NULLIF(Total_Sales, 0), 2
    ) AS Health_Wellness_Sale_Percent     
FROM Category_Sales
ORDER BY
    Generation;  
```

----
