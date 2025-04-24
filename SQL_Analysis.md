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
ORDER BY Receipts_Scanned DESC

---
**## 2. 🧬 Health & Wellness Sales Percentage by Generation
**This query classifies users into generational cohorts (Gen Z, Millennials, Gen X, Boomers) based on their birth year, and calculates what percentage of each generation's total spend went toward the Health & Wellness category.

sql
Copy
Edit
WITH Generations AS (
    SELECT
        u.ID AS USER_ID,
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
        SUM(t.FINAL_SALE) AS Total_Sales,
        SUM(CASE WHEN p.CATEGORY_1 = 'Health & Wellness' THEN t.FINAL_SALE ELSE 0 END)
            AS Health_Wellness_sales
    FROM dbo.transactions_cleaned AS t
    INNER JOIN Generations g ON t.USER_ID = g.USER_ID
    INNER JOIN dbo.products_cleaned p ON t.BARCODE = p.BARCODE
    WHERE t.FINAL_SALE IS NOT NULL
    GROUP BY g.Generation
)
SELECT
    Generation,
    ROUND(CAST(Health_Wellness_sales AS DECIMAL(18,2)) * 100.0 / NULLIF(Total_Sales, 0), 2)
        AS Health_Wellness_Sale_Percent
FROM Category_Sales
ORDER BY Generation;

---
