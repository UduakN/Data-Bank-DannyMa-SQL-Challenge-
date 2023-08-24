# Data-Bank-DannyMa-SQL-Challenge-

![image](https://github.com/UduakN/Data-Bank-DannyMa-SQL-Challenge-/assets/128192166/8f433710-2c86-4d38-bee8-ff0de7b0fade)

# INTRODUCTION
In this case study, Danny thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world beacuse there is a new innovation in the financial industry called Neo-Banks: new aged digital only banks without physical branches,so he decides to launch a new initiative - Data Bank!

Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!

Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. There are a few interesting caveats that go with this business model, and this is where the Data Bank team need your help!

The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need.

This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!

# ABOUT THE DATA

This database consists of three tables.

•	Regions – This table is made up of 2 columns and 5 rows.

•	Customer_transactions – This table is made up of 4 columns and 5439 rows.

•	Customer_nodes – This table is made up of 5 columns and 4500 rows.


# Entity Relationship diagram


![image](https://github.com/UduakN/Data-Bank-DannyMa-SQL-Challenge-/assets/128192166/852e6ab1-d720-4f05-a6c2-16359132b328)

# CASE STUDY QUESTIONS AND INSIGHTS

# A. Customer Nodes Exploration

# 1. How many unique nodes are there on the Data Bank system?

SELECT COUNT(DISTINCT node_id) AS uniquenodes
FROM customer_nodes;

![image](https://github.com/UduakN/Data-Bank-DannyMa-SQL-Challenge-/assets/128192166/03aa9a98-359a-4da2-9622-0e71ed104cf0)

# 2. What is the number of nodes per region?

SELECT r.region_id, r.region_name, COUNT(n.node_id) AS nodes
FROM customer_node n 
JOIN regions r
  ON n.region_id = r.region_id
GROUP BY r.region_id, r.region_name
ORDER BY r.region_id;

![image](https://github.com/UduakN/Data-Bank-DannyMa-SQL-Challenge-/assets/128192166/185eb2c9-818f-4a29-82bd-4171f4d6ac98)



# 3. How many customers are allocated to each region?

SELECT r.region_id,r.region_name,COUNT(DISTINCT n.customer_id) AS customers
FROM customer_nodes n
JOIN regions r
  ON n.region_id = r.region_id
GROUP BY r.region_id, r.region_name
ORDER BY r.region_id;

![image](https://github.com/UduakN/Data-Bank-DannyMa-SQL-Challenge-/assets/128192166/a6039cbc-0921-4954-858d-0d2fd1b6a36b)

# 4. How many days on average are customers reallocated to a different node?

WITH customerDates AS (
  SELECT customer_id,region_id,node_id,MIN(start_date) AS firstdate
  FROM customer_nodes
  GROUP BY customer_id, region_id, node_id
),
reallocation AS (
  SELECT customer_id,node_id,region_id,firstdate,
DATEDIFF(DAY, firstdate, 
             LEAD(firstdate) OVER(PARTITION BY customer_id 
                                   ORDER BY firstdate)) AS movingdays
  FROM customerDates
)

SELECT 
  AVG(CAST(movingdays AS FLOAT)) AS avg_movingdays
FROM reallocation;

![image](https://github.com/UduakN/Data-Bank-DannyMa-SQL-Challenge-/assets/128192166/d8e21d1a-f035-46c3-a676-07bfd4eaa008)

# 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

WITH customerDates AS (
  SELECT 
    customer_id,
    region_id,
    node_id,
    MIN(start_date) AS first_date
  FROM customer_nodes
  GROUP BY customer_id, region_id, node_id
),
reallocation AS (
  SELECT
    customer_id,
    region_id,
    node_id,
    first_date,
    DATEDIFF(DAY, first_date, LEAD(first_date) OVER(PARTITION BY customer_id ORDER BY first_date)) AS moving_days
  FROM customerDates
)

SELECT 
  DISTINCT r.region_id,
  rg.region_name,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY r.moving_days) OVER(PARTITION BY r.region_id) AS median,
  PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY r.moving_days) OVER(PARTITION BY r.region_id) AS percentile_80,
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY r.moving_days) OVER(PARTITION BY r.region_id) AS percentile_95
FROM reallocation r
JOIN regions rg ON r.region_id = rg.region_id
WHERE moving_days IS NOT NULL;

![image](https://github.com/UduakN/Data-Bank-DannyMa-SQL-Challenge-/assets/128192166/fd354ae9-745e-4589-81d6-0ee30fda1f36)

# B. Customer Transactions

# 1. What is the unique count and total amount for each transaction type?

SELECT txn_type, COUNT(*) AS unique_count, SUM(txn_amount) AS total_amount
FROM customer_transactions
GROUP BY txn_type;

![image](https://github.com/UduakN/Data-Bank-DannyMa-SQL-Challenge-/assets/128192166/fe219914-6f71-48eb-99f4-d060a5b4803e)


# 2. What is the average total historical deposit counts and amounts for all customers?

WITH customerDeposit AS (
  SELECT customer_id,txn_type,COUNT(*) AS dep_count,SUM(txn_amount) AS dep_amount
  FROM customer_transactions
  WHERE txn_type = 'deposit'
  GROUP BY customer_id, txn_type
)

SELECT
  AVG(dep_count) AS avg_dep_count,
  AVG(dep_amount) AS avg_dep_amount
FROM customerDeposit;

![image](https://github.com/UduakN/Data-Bank-DannyMa-SQL-Challenge-/assets/128192166/cf97dca6-6a73-40ae-af02-4552267f39c2)

# 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

WITH cte_transaction AS (
  SELECT 
    customer_id,
    MONTH(txn_date) AS months,
    SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit_count,
    SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_count,
    SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal_count
  FROM customer_transactions
  GROUP BY customer_id, MONTH(txn_date)
)

SELECT 
  months,
  COUNT(customer_id) AS customer_count
FROM cte_transaction
WHERE deposit_count > 1
  AND (purchase_count = 1 OR withdrawal_count = 1)
GROUP BY months;

![image](https://github.com/UduakN/Data-Bank-DannyMa-SQL-Challenge-/assets/128192166/e4b309fe-f674-4808-88f8-36f37871802a)


# 4. What is the closing balance for each customer at the end of the month?

DECLARE @maxDate DATE;
SET @maxDate = (SELECT EOMONTH(MAX(txn_date)) FROM customer_transactions);

--CTE 1: Monthly transactions of each customer
WITH monthly_transactions AS (
  SELECT
    customer_id,
    EOMONTH(txn_date) AS end_date,
    SUM(CASE WHEN txn_type IN ('withdrawal', 'purchase') THEN -txn_amount
             ELSE txn_amount END) AS transactions
  FROM customer_transactions
  GROUP BY customer_id, EOMONTH(txn_date)
),

--CTE 2: Increment last days of each month till they are equal to @maxDate 
recursive_dates AS (
  SELECT
    DISTINCT customer_id,
    CAST('2020-01-31' AS DATE) AS end_date
  FROM customer_transactions
  UNION ALL
  SELECT 
    customer_id,
    EOMONTH(DATEADD(MONTH, 1, end_date)) AS end_date
  FROM recursive_dates
  WHERE EOMONTH(DATEADD(MONTH, 1, end_date)) <= @maxDate
)

SELECT 
  r.customer_id,
  r.end_date,
  COALESCE(m.transactions, 0) AS transactions,
  SUM(m.transactions) OVER (PARTITION BY r.customer_id ORDER BY r.end_date 
      ROWS UNBOUNDED PRECEDING) AS closing_balance
FROM recursive_dates r
LEFT JOIN  monthly_transactions m
  ON r.customer_id = m.customer_id
  AND r.end_date = m.end_date;

  ![image](https://github.com/UduakN/Data-Bank-DannyMa-SQL-Challenge-/assets/128192166/88e794fb-40a1-43d6-b121-42babf03e218)

# 5. What is the percentage of customers who increase their closing balance by more than 5%?

--End date in the month of the max date of our dataset (Q4)
DECLARE @maxDate DATE;
SET @maxDate = (SELECT EOMONTH(MAX(txn_date)) FROM customer_transactions);

--CTE 1: Monthly transactions of each customer (Q4)
WITH monthly_transactions AS (
  SELECT
    customer_id,
    EOMONTH(txn_date) AS end_date,
    SUM(CASE WHEN txn_type IN ('withdrawal', 'purchase') THEN -txn_amount
             ELSE txn_amount END) AS transactions
  FROM customer_transactions
  GROUP BY customer_id, EOMONTH(txn_date)
),

--CTE 2: Increment last days of each month till they are equal to @maxDate (Q4)
recursive_dates AS (
  SELECT
    DISTINCT customer_id,
    CAST('2020-01-31' AS DATE) AS end_date
  FROM customer_transactions
  UNION ALL
  SELECT 
    customer_id,
    EOMONTH(DATEADD(MONTH, 1, end_date)) AS end_date
  FROM recursive_dates
  WHERE EOMONTH(DATEADD(MONTH, 1, end_date)) <= @maxDate
),

-- CTE 3: Closing balance of each customer by monthly (Q4)
customers_balance AS (
  SELECT 
    r.customer_id,
    r.end_date,
    COALESCE(m.transactions, 0) AS transactions,
    SUM(m.transactions) OVER (PARTITION BY r.customer_id ORDER BY r.end_date 
        ROWS UNBOUNDED PRECEDING) AS closing_balance
    FROM recursive_dates r
    LEFT JOIN  monthly_transactions m
      ON r.customer_id = m.customer_id
      AND r.end_date = m.end_date
),

--CTE 4: CTE 3 & next_balance
customers_next_balance AS (
  SELECT *,
    LEAD(closing_balance) OVER(PARTITION BY customer_id ORDER BY end_date) AS next_balance
  FROM customers_balance
),

--CTE 5: Calculate the increase percentage of closing balance for each customer
pct_increase AS (
  SELECT *,
    100.0*(next_balance-closing_balance)/closing_balance AS pct
  FROM customers_next_balance
  WHERE closing_balance ! = 0 AND next_balance IS NOT NULL
)

--Create a temporary table because of the error: Null value is eliminated by an aggregate or other SET operation
SELECT *
INTO #temp
FROM pct_increase;

--Calculate the percentage of customers whose closing balance increasing 5% compared to the previous month
SELECT CAST(100.0*COUNT(DISTINCT customer_id) AS FLOAT)
      / (SELECT COUNT(DISTINCT customer_id) FROM customer_transactions) AS pct_customers
FROM #temp
WHERE pct > 5;

![image](https://github.com/UduakN/Data-Bank-DannyMa-SQL-Challenge-/assets/128192166/2e6f0516-0a12-4512-a71c-078d29173c34)


















