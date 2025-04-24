# SQL Analysis: Fetch Dataset Insights(Closed Ended Questions)

This file contains annotated SQL queries used to analyze the Fetch Takehome dataset. All queries were developed and tested using ****SQL Server Management Studio (SSMS).****

---

## 1. Finding Power Users
**Goal:** 

-- Here, the main goal is to identify truly influential and valuable users. I defined those users as those who not only as big sales but also are in the top 90% percentile in the receipt_counts. With this approach, I aimed to get both big shoppers driving revenue and engagement shoppers as well.

**Approach**

-- First, I will be determining the total distinct receipts and total sales by users. The I will be putting a  cut-off for the high-engagement, which will be  the 90th percentile of receipt counts (top 10%) and in the end, I will be filtering  out the  users at who are above the cut-off or on same level and order by the Total Sales, highlighting the top 10 users.
```
/* -----------------------------------------------------------------------
   POWER USERS
   • Engagement: Top 10% by distinct receipts (Pct90)
   • Value:    Among those, Top 10 by Total_Sales
------------------------------------------------------------------------
*/

--Calculating of the User Metrics
WITH UserActivity AS (
    SELECT
        u.ID                            AS USER_ID,
        COUNT(DISTINCT t.RECEIPT_ID)    AS Receipt_Count,
        SUM(t.FINAL_SALE)               AS Total_Sales
    FROM dbo.transactions_cleaned AS t
    JOIN dbo.users_cleaned       AS u ON t.USER_ID = u.ID
    WHERE t.FINAL_SALE IS NOT NULL
    GROUP BY u.ID
),
-- Computing the 90th percentile threshold
Percentile_90 AS (                   
    SELECT 
        PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY Receipt_Count) 
        OVER () AS Percent_90
    FROM UserActivity
),
-- Filtering of users above or at the defined threshold level
Filtered_Users AS (              
    SELECT DISTINCT ua.USER_ID, ua.Receipt_Count, ua.Total_Sales
    FROM UserActivity ua
    CROSS JOIN Percentile_90
    WHERE ua.Receipt_Count >= Percentile_90.Percent_90
)
-- Return Top 10 by Total_Sales
SELECT TOP 10              
    USER_ID,
    Receipt_Count,
    Total_Sales
FROM Filtered_Users
ORDER BY Total_Sales DESC;
```

## 2. Leading Brand in Dips & Salsa
**Goal:**

-- Finding the brand which has the lead, i.e, the largest share of sales in the category of “Dips & Salsa” segment, for which I assumed during the most recent year with data.

**Approach**

-- First I normalized the category text (trim + upper) to avoid mismatches, post that I went for identifying the latest year that actually contains any “Dips & Salsa” sales and have aggregate total sales by brand for that particular year and have computed market share (%) relative to the segment total sales of all brand in that year. In the last, I have returned only 1 value as I am aiming for only the top 1 brand by market share.

```
* --------------------------------------------------------------
   Leading brand in Dips & Salsa – robust to case/spacing and
   chooses the latest year that contains any Dips & Salsa sales
   -------------------------------------------------------------- */

DECLARE @LatestYear INT =
(
    SELECT MAX(YEAR(t.PURCHASE_DATE))
    FROM   dbo.transactions_cleaned t
    JOIN   dbo.products_cleaned    p ON t.BARCODE = p.BARCODE
    WHERE  t.FINAL_SALE IS NOT NULL
      AND  UPPER(LTRIM(RTRIM(p.CATEGORY_2))) = 'DIPS & SALSA'
);

WITH CategorySales AS (
    SELECT
        p.BRAND,
        SUM(t.FINAL_SALE) AS Brand_Sales
    FROM   dbo.transactions_cleaned t
    JOIN   dbo.products_cleaned    p ON t.BARCODE = p.BARCODE
    WHERE  t.FINAL_SALE IS NOT NULL
      AND  UPPER(LTRIM(RTRIM(p.CATEGORY_2))) = 'DIPS & SALSA'
      AND  YEAR(t.PURCHASE_DATE) = @LatestYear
    GROUP BY p.BRAND
),
-- Calculating total_sales of segment
TotalSegment AS (
    SELECT SUM(Brand_Sales) AS Segment_Sales FROM CategorySales)

-- Brand with highest segment revenue
SELECT TOP 1
    BRAND,
    Brand_Sales,
    ROUND(Brand_Sales * 100.0 /
          NULLIF((SELECT SUM(Brand_Sales) FROM CategorySales),0), 2) AS Market_Share_Percentage
FROM CategorySales
ORDER BY Brand_Sales DESC;

```

## 3. Year-Over-Year Sales Growth (%)
**Goal:** 

-- Here I aim to calculate the percentage of change between the two latest years(I will be going with fiscal because, as organization planning or resource allocation is done based on fiscal planning). 

**Approach** 

-- First, I group the transactions into a fiscal year label, which will be (Jul–Dec from the next calendar year, and Jan–Jun of the same year. Post this, I will be going with the Sum sales per fiscal year and then ranking fiscal years in descending order. After that, I will be using join for the most recent (rn = 1) with the prior year(rn = 2). For getting the metric of year-over-year Sales growth percentage, I will be calculating (CurrentFY − PriorFY) / PriorFY × 100.

```
/* --------------------------------------------------------------
   YoY growth – find the two latest fiscal years with sales
   -------------------------------------------------------------- */
-- Labeling of the transactions with the correct Fiscal Year. If the purchase was in July–December, it's part of next year's fiscal year.

WITH Fiscal_Label AS (
    SELECT
        CASE WHEN MONTH(PURCHASE_DATE) >= 7
             THEN YEAR(PURCHASE_DATE) + 1   
             ELSE YEAR(PURCHASE_DATE)      
        END  AS FiscalYear,
        FINAL_SALE
    FROM dbo.transactions_cleaned
    WHERE FINAL_SALE IS NOT NULL
),
SalesByFiscalYear AS (
    SELECT
        FiscalYear,
  -- Calculating the sum of total sales by each fiscal year and assigning descending row numbers
        SUM(FINAL_SALE) AS Fiscal_Sales,
        ROW_NUMBER() OVER (ORDER BY FiscalYear DESC) AS rn
    FROM Fiscal_Label
    GROUP BY FiscalYear
)
SELECT
    cur.FiscalYear           AS Current_Fiscal_Year,
    prev.FiscalYear          AS Prior_Fiscal_Year,
    cur.Fiscal_Sales             AS Current_Fiscal_Year_Sales,
    prev.Fiscal_Sales            AS Prior_Fiscal_Year_Sales,
-- Joining of the current Fiscal Year with the previous Fiscal Year and calculating YoY percentage growth      
     ROUND( (cur.Fiscal_Sales - prev.Fiscal_Sales) * 100.0 /
           NULLIF(prev.Fiscal_Sales,0), 2)      AS YoY_Growth_Percentage
FROM SalesByFiscalYear  cur
JOIN SalesByFiscalYear  prev
  ON prev.rn = cur.rn + 1       
WHERE cur.rn = 1;                
```

----
