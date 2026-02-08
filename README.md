# plsql_window_function_29571_Larissa

**Name:** KEZA INGABIRE Larissa

**ID:** 29571

 **Group A**

**Instructor:** Eric Maniraguha 

# **ASSIGNMENT: DATABASE PL/SQL**

## **Step 1:** Problem Definition!

**Business Context**

The company is a retail e-commerce company operating in the consumer goods industry. 
The analysis is conducted for the Sales and Marketing Department to improve revenue performance across regions. 

**Data Challenge** 

The company collects transactional sales data from multiple regions, but lacks clear insights into top-performing products, customer purchasing behavior, and sales trends over time. 
Management cannot easily identify inactive customers, products with no sales, or compare 
customer performance within regions. 

**Expected Outcome** 

The analysis aims to support data-driven decision-making by identifying high-value products, 
tracking sales trends, segmenting customers, and highlighting growth or decline patterns to 
improve marketing and inventory strategies.

## **Step 2**: Success Criteria (Window-Function-Driven Goals) 

The analysis must achieve exactly five measurable goals: 

1. Identify top 5 products per region using RANK() 
2. Calculate running monthly sales totals using SUM() OVER() 
3. Measure month-over-month sales growth using LAG() 
4. Segment customers into quartiles based on total spending using NTILE(4) 
5. Compute three-month moving averages of sales using AVG() OVER()

## **DATABSE SCHEMA** <img width="1080" height="1000" alt="Togetha db" src="https://github.com/user-attachments/assets/cbb64294-64d8-423c-8c47-0d9d76c0f65c" />

**AS FOLLOW:**

1.**Customer Table**

```sql
CREATE TABLE customers ( 
customer_id INT PRIMARY KEY, 
customer_name VARCHAR(100) NOT NULL, 
region VARCHAR(50), 
join_date DATE 
); 
```

 2. **Products Table**
```sql     
CREATE TABLE products ( 
product_id INT PRIMARY KEY, 
product_name VARCHAR(100) NOT NULL, 
    category VARCHAR(50), 
    price DECIMAL(10,2) 
); 
```
 
**Description:** 
 This table contains product details such as category and unit price. 
 It is referenced by the sales table to associate each transaction 
with a product. 
 
**3. Sales Table**
```sql
CREATE TABLE sales ( 
    sale_id INT PRIMARY KEY, 
    customer_id INT, 
    product_id INT, 
    sale_date DATE, 
    quantity INT, 
    total_amount DECIMAL(10,2), 
    CONSTRAINT fk_customer 
        FOREIGN KEY (customer_id) 
        REFERENCES customers(customer_id), 
    CONSTRAINT fk_product 
        FOREIGN KEY (product_id) 
        REFERENCES products(product_id));

```
  **Description:**
  
This table records all sales transactions. 
It acts as a bridge table connecting customers and products through 
foreign keys. 
Schema Verification Queries 
The following queries can be used to display and verify the schema 
after creation. 

## **STEP 3:Business Justification**


This schema supports:

● JOIN operations across customers, products, and sales 

● Window function analysis such as ranking, running totals, and 
customer segmentation

● Clear identification of inactive customers and unsold products

#### **Step 4: Part A:SQL JOINs Implementation** 

**1. INNER JOIN**

**Purpose:** Retrieve transactions with valid customers and products. 
```sql
SELECT c.customer_name, p.product_name, s.total_amount 
FROM sales s 
INNER JOIN customers c ON s.customer_id = c.customer_id 
INNER JOIN products p ON s.product_id = p.product_id;
```

RESULTS:<img width="959" height="195" alt="screenshot 1" src="https://github.com/user-attachments/assets/ce6a2a76-d7ea-4f36-95d7-625766577726" />


**Business Interpretation:**

This query shows only completed transactions where both customer and product information 
exists. It helps management analyze valid sales performance.

**2. LEFT JOIN**
   
**Purpose:** Identify customers who have never made a transaction.
```sql
SELECT c.customer_name 
FROM customers c 
LEFT JOIN sales s ON c.customer_id = s.customer_id 
WHERE s.sale_id IS NULL;
```

RESULTS:<img width="960" height="97" alt="screenshot 3" src="https://github.com/user-attachments/assets/fa888587-c3eb-44e0-8878-23d4850dd879" />



**Business Interpretation:**

These customers represent inactive users who may be targeted with promotions or 
re-engagement campaigns.

**3. RIGHT JOIN**


Purpose: Detect products with no sales activity. 

```sql
SELECT p.product_name 
FROM sales s 
RIGHT JOIN products p ON s.product_id = p.product_id 
WHERE s.sale_id IS NULL;
```

**RESULTS:** <img width="960" height="111" alt="screenshot 4" src="https://github.com/user-attachments/assets/f20e1e84-2db0-445e-ab79-745c4d92a484" />


**Business Interpretation:**

Products without sales may require price adjustments, better marketing, or discontinuation.

**4. FULL OUTER JOIN** 

**Purpose:** Compare customers and products including unmatched records. 
```sql
SELECT c.customer_name, p.product_name 
FROM customers c 
FULL OUTER JOIN products p 
ON c.region = p.category; 
```
**RESULTS:** <img width="960" height="276" alt="Full join screenshot" src="https://github.com/user-attachments/assets/69868abc-1db5-4731-82bf-e90b2422c618" />



**Business Interpretation:**

This query highlights mismatches between customer regions and product categories, 
supporting strategic alignment.

**5. SELF JOIN**


**Purpose:** Compare customers within the same region. 
```sql
SELECT a.customer_name AS customer1, b.customer_name AS customer2,a.region 
FROM customers a 
JOIN customers b 
ON a.region = b.region 
AND a.customer_id <> b.customer_id; 
```
RESULTS:<img width="960" height="122" alt="SELF OUTER JOIN" src="https://github.com/user-attachments/assets/2a974227-0698-4e0b-9b69-464e38959413" />


**Business Interpretation:**

This comparison helps identify customers operating in the same region for peer performance 
analysis.

##### **Step 5: Part B:Window Functions Implementation**

**1. Ranking Functions**


**Use case:** Top customers by revenue. 
```sql
SELECT customer_id, 
SUM(total_amount) AS total_spent, 
RANK() OVER (ORDER BY SUM(total_amount) DESC) AS rank_position 
FROM sales 
GROUP BY customer_id;
```
**RESULTS:** <img width="960" height="198" alt="Ranking F SCREENSHOOT" src="https://github.com/user-attachments/assets/aeafdbad-e184-4e5e-b4ea-548204930fcb" />

**Interpretation:**

Customers are ranked by total spending, enabling identification of high-value clients.

**2. Aggregate Window Functions** 


**Use case:** Running monthly sales totals. 
```sql
SELECT sale_date, 
       SUM(total_amount) OVER ( 
           ORDER BY sale_date 
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW 
       ) AS running_total 
FROM sales; 
```
**RESULTS:** <img width="960" height="189" alt="Aggregation W FUNCTION ST" src="https://github.com/user-attachments/assets/8dd08404-cf22-4cce-8cbc-99df2ad70a1e" />

 **Interpretation:** 

 This shows cumulative revenue growth over time, supporting trend analysis.

 **3. Navigation Functions** 
 

**Use case:** Month-over-month sales growth. 
```sql
SELECT sale_date, 
       total_amount, 
       total_amount - LAG(total_amount) OVER (ORDER BY sale_date) AS growth 
FROM sales; 
```
**RESULTS:** <img width="960" height="170" alt="Navigation F Screenshot" src="https://github.com/user-attachments/assets/a09ed6de-311e-4747-a926-6de04546a480" />

**Interpretation:**

 Positive growth indicates increasing sales, while negative values highlight declining 
performance. 

**4. Distribution Functions** 


**Use case:** Customer segmentation. 
```sql
SELECT customer_id, 
       NTILE(4) OVER (ORDER BY SUM(total_amount)) AS spending_quartile 
FROM sales 
GROUP BY customer_id; 
```
 RESULTS:<img width="960" height="169" alt="Distribution function screenshot" src="https://github.com/user-attachments/assets/3ed5a7b3-c505-4456-be83-e89b3a955e45" />

**Interpretation:**

Customers are grouped into quartiles, enabling targeted marketing strategies.

## **Step 7: Results Analysis** 

**Descriptive Analysis**

Sales are concentrated among a small number of customers and products, with noticeable 
regional differences. 

**Diagnostic Analysis** 

Inactive customers and unsold products reduce overall revenue potential due to poor 
engagement and inventory misalignment. 

**Prescriptive Analysis** 

The company should focus on customer retention campaigns, discontinue low-performing 
products, and prioritize top-ranked products per region.


## **Step 8: References**

● Oracle SQL Documentation 

● PostgreSQL Window Functions Guide 

● W3Schools SQL JOINs Tutorial


##  **Academic Integrity Statement**

All sources were properly cited.

Implementations and analysis represent original work. 

**Signature:** KEZA I. Larissa 

**Date:** February 07, 2025



## 📧 Contact

*Email:* Lakeza06@gmail.com 

*GitHub:* github.com/Kezalarissa
