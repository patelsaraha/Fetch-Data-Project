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
