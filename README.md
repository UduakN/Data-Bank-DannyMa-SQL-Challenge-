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






