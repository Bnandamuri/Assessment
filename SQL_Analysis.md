# SQL Analysis: Fetch Dataset Insights

This file contains annotated SQL queries used to analyze the Fetch Takehome dataset. All queries were developed and tested using ****SQL Server Management Studio (SSMS).****

---

## 1. Top 5 Brands by Receipts Scanned (Age ≥ 21)
This query finds the top 5 brands based on the number of distinct receipts scanned by users who are 21 or older.

```SELECT TOP 5
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
ORDER BY Receipts_Scanned DESC;

---
****## 2. Top 5 Sales by Brands Among Users (Account ≥ 6 Months)**
**This query finds the top 5 brands by total sales, filtered to include only users who have been active for at least 6 months (based on the difference between account creation and purchase date).

```-- Calculating total sales per brand and ranking them
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
    -- Filtering out the null sales and ensuring user has had account ≥ 6 months
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


