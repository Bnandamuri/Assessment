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

**## 2.  Top 5 Brands by Receipts Scanned (Age ≥ 21)**


