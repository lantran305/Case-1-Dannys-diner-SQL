# Week 1: Danny's Diner 🍣

## 1. Business Context

Danny's Diner is a restaurant serving three popular Japanese dishes: sushi, curry, and ramen. Danny wants to better understand **customer purchasing behavior**, spending patterns, and menu preferences to **improve customer loyalty** and deliver a more **personalized dining experience**.

This project uses SQL to analyze transaction data and generate insights to support customer relationship management and loyalty program decisions.

## 2. Objectives

- Calculate total spending and visit frequency for each customer.
- Identify customers' first purchases and most popular menu items.
- Analyze purchasing behavior before and after joining the loyalty program.
- Calculate customer loyalty points under different reward rules.
- Create customer-level datasets to support further analysis.

## 3. Dataset

The dataset is provided by the [8 Week SQL Challenge – Case Study #1](https://8weeksqlchallenge.com/case-study-1/).

| Table | Description |
|---|---|
| `sales` | Customer transactions, including order dates and product IDs |
| `menu` | Menu items, product names, and prices |
| `members` | Customer loyalty program membership dates |

The dataset contains transaction records for three customers, two of whom joined the loyalty program.

## 4. Tools

- PostgreSQL
- VS Code
- pgAdmin

## 5. SQL Techniques

- Data exploration and aggregation
- Table joins using `INNER JOIN` and `LEFT JOIN`
- Aggregate functions: `SUM`, `COUNT`
- Common Table Expressions (CTEs)
- Window functions: `RANK`, `DENSE_RANK`, `ROW_NUMBER`
- Conditional logic using `CASE WHEN`
- Date comparisons and date-based filtering
- Customer segmentation and loyalty points calculations

## 6. Examples of Business Questions & Queries

### A. Customer Spending and Visits


```sql
-- 1. Tổng số tiền mỗi khách hàng đã chi.
SELECT
    s.customer_id,
    SUM(m.price) AS total_spent
FROM sales s
JOIN menu m ON m.product_id = s.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
    
```

<img width="360" height="155" alt="image" src="https://github.com/user-attachments/assets/2a42a3b6-e632-44d7-bc4c-d006e697d8ae" />


```sql
-- 3. Món đầu tiên mỗi khách hàng mua.
WITH ranked_sales AS (
    SELECT
        s.customer_id,
        m.product_name,
        s.order_date,
        ROW_NUMBER() OVER (
            PARTITION BY s.customer_id
            ORDER BY s.order_date
        ) AS purchase_rank
    FROM sales s
    JOIN menu m ON m.product_id = s.product_id
)
SELECT customer_id, product_name AS first_item
FROM ranked_sales
WHERE purchase_rank = 1
ORDER BY customer_id;
    
```

<img width="389" height="181" alt="image" src="https://github.com/user-attachments/assets/025798b3-cd0c-4c62-9d99-382a98dc163b" />



### B. Menu Preferences

```sql
-- 4. Món được mua nhiều nhất và tổng số lần mua.
WITH item_counts AS (
    SELECT
        m.product_name,
        COUNT(*) AS purchase_count
    FROM sales s
    JOIN menu m ON m.product_id = s.product_id
    GROUP BY m.product_name
),
ranked_items AS (
    SELECT
        product_name,
        purchase_count,
        RANK() OVER (ORDER BY purchase_count DESC) AS item_rank
    FROM item_counts
)
SELECT product_name, purchase_count
FROM ranked_items
WHERE item_rank = 1;
    
```

<img width="373" height="111" alt="image" src="https://github.com/user-attachments/assets/49c44c47-1903-4469-acba-229608908689" />

```sql
-- 5. Món được mua nhiều nhất của từng khách hàng (giữ lại mọi trường hợp đồng hạng).
WITH customer_item_counts AS (
    SELECT
        s.customer_id,
        m.product_name,
        COUNT(*) AS purchase_count
    FROM sales s
    JOIN menu m ON m.product_id = s.product_id
    GROUP BY s.customer_id, m.product_name
),
ranked_customer_items AS (
    SELECT
        customer_id,
        product_name,
        purchase_count,
        RANK() OVER (
            PARTITION BY customer_id
            ORDER BY purchase_count DESC
        ) AS item_rank
    FROM customer_item_counts
)
SELECT customer_id, product_name, purchase_count
FROM ranked_customer_items
WHERE item_rank = 1
ORDER BY customer_id, product_name;
    
```

<img width="505" height="196" alt="image" src="https://github.com/user-attachments/assets/b6632e96-324e-401e-a1df-8ac91d562b2b" />


### C. Loyalty Program Analysis

```sql
-- 6. Món đầu tiên khách hàng mua sau khi trở thành thành viên.
WITH ranked_member_sales AS (
    SELECT
        s.customer_id,
        m.product_name,
        s.order_date,
        ROW_NUMBER() OVER (
            PARTITION BY s.customer_id
            ORDER BY s.order_date
        ) AS purchase_rank
    FROM sales s
    JOIN members mb
        ON mb.customer_id = s.customer_id
       AND s.order_date >= mb.join_date
    JOIN menu m ON m.product_id = s.product_id
)
SELECT customer_id, product_name AS first_item_after_join
FROM ranked_member_sales
WHERE purchase_rank = 1
ORDER BY customer_id;
    
```
<img width="399" height="157" alt="image" src="https://github.com/user-attachments/assets/38149830-7da4-4de8-b451-f2219a993359" />

```sql
-- 10. Tổng điểm đến hết tháng 1/2021, cộng điểm 2x cho mọi món trong
---tuần đầu tiên sau khi gia nhập (tính cả ngày gia nhập).
SELECT
    mb.customer_id,
    SUM(
        m.price * CASE
            WHEN m.product_name = 'sushi'
              OR (
                  s.order_date >= mb.join_date
                  AND s.order_date < mb.join_date + 7
              )
            THEN 20
            ELSE 10
        END
    ) AS total_points
FROM members mb
JOIN sales s
    ON s.customer_id = mb.customer_id
   AND s.order_date < DATE '2021-02-01'
JOIN menu m ON m.product_id = s.product_id
GROUP BY mb.customer_id
ORDER BY mb.customer_id;
    
```
<img width="348" height="133" alt="image" src="https://github.com/user-attachments/assets/8f84526a-381e-405b-88bf-ee7036504cff" />


