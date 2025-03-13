#Initial Observations on Data Quality:

#Users Table (USER_TAKEHOME.csv):

-  BIRTH_DATE, STATE, LANGUAGE, and GENDER have missing values.
-  CREATED_DATE and BIRTH_DATE are stored as object types instead of dates.

#Transactions Table (TRANSACTION_TAKEHOME.csv):

-  BARCODE has missing values (~11.5% missing).
-  FINAL_QUANTITY and FINAL_SALE are stored as object types instead of numerical values.
-  PURCHASE_DATE and SCAN_DATE should be converted to date format.

#Products Table (PRODUCTS_TAKEHOME.csv):

-  CATEGORY_3, CATEGORY_4, MANUFACTURER, and BRAND have missing values.
-  BARCODE is stored as a float but should be treated as a string.# Fetch-Data-Project

#Additional Data Quality Findings:

#Duplicates:

-  No duplicate records in the Users table.
-  Transactions table has 171 duplicate records.
-  Products table has 215 duplicate records.
-  Unique Values:

-  Users table has 100,000 unique users.
-  Transactions table has 24,440 unique receipt IDs, meaning multiple transactions can be associated with the same receipt.
-  Products table has 841,342 unique barcodes, but the number of barcodes in transactions is much lower.
-  Barcode Mismatch:

-  The transactions table has 830,315 fewer unique barcodes than the products table. This suggests either missing barcode information in transactions or additional products in the products dataset that never appeared in transactions.



**Second: provide SQL queries**

**Closed-ended questions:**

What are the top 5 brands by receipts scanned among users 21 and over?

WITH AgeFilteredUsers AS (
    SELECT ID, BIRTH_DATE
    FROM Users
    WHERE DATE_PART('year', CURRENT_DATE) - DATE_PART('year', BIRTH_DATE) >= 21
)
SELECT p.BRAND, COUNT(DISTINCT t.RECEIPT_ID) AS receipt_count
FROM Transactions t
JOIN AgeFilteredUsers u ON t.USER_ID = u.ID
JOIN Products p ON t.BARCODE = p.BARCODE
GROUP BY p.BRAND
ORDER BY receipt_count DESC
LIMIT 5;

What are the top 5 brands by sales among users that have had their account for at least six months?

WITH EligibleUsers AS (
    SELECT ID
    FROM Users
    WHERE CREATED_DATE <= CURRENT_DATE - INTERVAL '6 months'
)
SELECT p.BRAND, SUM(t.FINAL_SALE) AS total_sales
FROM Transactions t
JOIN EligibleUsers u ON t.USER_ID = u.ID
JOIN Products p ON t.BARCODE = p.BARCODE
GROUP BY p.BRAND
ORDER BY total_sales DESC
LIMIT 5;

What is the percentage of sales in the Health & Wellness category by generation?

WITH UserGenerations AS (
    SELECT ID,
           CASE 
               WHEN DATE_PART('year', CURRENT_DATE) - DATE_PART('year', BIRTH_DATE) >= 58 THEN 'Boomers'
               WHEN DATE_PART('year', CURRENT_DATE) - DATE_PART('year', BIRTH_DATE) BETWEEN 42 AND 57 THEN 'Gen X'
               WHEN DATE_PART('year', CURRENT_DATE) - DATE_PART('year', BIRTH_DATE) BETWEEN 26 AND 41 THEN 'Millennials'
               WHEN DATE_PART('year', CURRENT_DATE) - DATE_PART('year', BIRTH_DATE) <= 25 THEN 'Gen Z'
           END AS Generation
    FROM Users
)
SELECT g.Generation, 
       SUM(t.FINAL_SALE) * 100.0 / (SELECT SUM(FINAL_SALE) FROM Transactions) AS sales_percentage
FROM Transactions t
JOIN UserGenerations g ON t.USER_ID = g.ID
JOIN Products p ON t.BARCODE = p.BARCODE
WHERE p.CATEGORY_1 = 'Health & Wellness'
GROUP BY g.Generation
ORDER BY sales_percentage DESC;




**Open-ended questions: for these, make assumptions and clearly state them when answering the question.**

Who are Fetch’s power users?

SELECT t.USER_ID, COUNT(DISTINCT t.RECEIPT_ID) AS receipt_count, SUM(t.FINAL_SALE) AS total_spent
FROM Transactions t
GROUP BY t.USER_ID
HAVING COUNT(DISTINCT t.RECEIPT_ID) > 50 AND SUM(t.FINAL_SALE) > 1000
ORDER BY total_spent DESC, receipt_count DESC;

Which is the leading brand in the Dips & Salsa category?

SELECT p.BRAND, SUM(t.FINAL_SALE) AS total_sales
FROM Transactions t
JOIN Products p ON t.BARCODE = p.BARCODE
WHERE p.CATEGORY_2 = 'Dips & Salsa'
GROUP BY p.BRAND
ORDER BY total_sales DESC
LIMIT 1;


At what percent has Fetch grown year over year?

WITH YearlySales AS (
    SELECT DATE_PART('year', PURCHASE_DATE) AS sales_year, SUM(FINAL_SALE) AS total_sales
    FROM Transactions
    GROUP BY sales_year
)
SELECT s1.sales_year AS year, 
       ((s1.total_sales - s2.total_sales) / s2.total_sales) * 100 AS growth_percentage
FROM YearlySales s1
JOIN YearlySales s2 ON s1.sales_year = s2.sales_year + 1
ORDER BY s1.sales_year DESC;

**Third: communicate with stakeholders**
**Example**

Hi [Stakeholder’s Name],

I wanted to share some key findings from my analysis of the Fetch data, including data quality concerns, an interesting trend, and a few outstanding questions that require further clarification.

Key Data Quality Issues:
Missing Data:
Some users have missing values in STATE, LANGUAGE, GENDER, and BIRTH_DATE, which impacts segmentation and demographic analysis.
The BARCODE field is missing in ~11.5% of transactions, making it difficult to attribute sales to specific brands.
Data Consistency Issues:
FINAL_SALE and FINAL_QUANTITY were stored as text instead of numeric values, requiring conversion.
Date fields (PURCHASE_DATE, SCAN_DATE, etc.) were inconsistent in format, which could lead to errors in time-based reporting.
Potential Data Gaps:
There are 830,315 fewer unique barcodes in transactions compared to the products table. This could indicate missing transaction records or product catalog inconsistencies.
Interesting Trend:
Top Brands by Receipts Scanned (Users 21+): A key finding from the data is that Fetch users aged 21 and over frequently scan receipts for [Brand X, Brand Y, Brand Z]. This suggests strong brand engagement and potential marketing opportunities for these brands.
Outstanding Questions & Request for Action:
Barcode Matching Issue: Can we confirm whether the product catalog is up to date and aligned with transaction records? Do we expect products to appear in transactions that are missing from our product master file?
Missing User Data: Is there a source where we can retrieve missing demographic information, or should we exclude incomplete records from certain analyses?
Growth Metrics Validation: The year-over-year growth calculation assumes all transaction records are captured. Can we verify whether any historical data is missing?
Would love to discuss these findings and any additional context you might have. Let me know if we can sync up to resolve these issues and refine the insights further.
