# SQL-Data-Exploration
Week 1
# 8-Week SQL Challenge




# 📕 Table of Contents
- [🛠️ Problem Statement](https://github.com/adunoluwa1/SQL-8-Weeks-Challenge/edit/main/Week_1/README.md#%EF%B8%8F-problem-statement)
- [📂 Dataset](https://github.com/adunoluwa1/SQL-8-Weeks-Challenge/edit/main/Week_1/README.md#-dataset)
- [✒️ Case Study Questions](https://github.com/adunoluwa1/SQL-8-Weeks-Challenge/edit/main/Week_1/README.md#%EF%B8%8Fcase-study-questions) 
- [🏆 Solutions](https://github.com/adunoluwa1/SQL-8-Weeks-Challenge/edit/main/Week_1/README.md#-solutions)

# 🛠️ Problem Statement
> Danny aims to leverage customer data to gain valuable insights into their behavior, preferences, and loyalty. Specifically, he seeks to understand:

- Visiting patterns and frequency
- Total spend and average transaction value
- Favorite menu items and purchasing habits

> By gaining a deeper understanding of his customers, Danny plans to enhance their experience through personalized offerings and targeted marketing initiatives. Furthermore, these insights will inform his decision on expanding the existing customer loyalty program.

> To facilitate this analysis, Danny has provided a sample dataset, which will be used to develop fully functional SQL queries. These queries will enable Danny's team to easily inspect the data and gain actionable insights without requiring extensive SQL knowledge.

# 📂 Dataset
This case study has 3 key datasets 
- sales
  <details><summary>View table</summary>
  <p>
  
    | customer_id  |     order_date      | product_id |
    |--------------|---------------------|------------|
    |       A      |     2021-01-01      |     1      |
    |       A      |     2021-01-01      |     2      |
    |       A      |     2021-01-07      |     2      |
    |       A      |     2021-01-10      |     3      |
    |       A      |     2021-01-11      |     3      |
    |       A      |     2021-01-11      |     3      |
    |       B      |     2021-01-01      |     2      |
    |       B      |     2021-01-02      |     2      |
    |       B      |     2021-01-04      |     1      |
    |       B      |     2021-01-11      |     1      |
    |       B      |     2021-01-16      |     3      |
    |       B      |     2021-02-01      |     3      |
    |       C      |     2021-01-01      |     3      |
    |       C      |     2021-01-01      |     3      |
    |       C      |     2021-01-07      |     3      |
  
  </p>
  </details>

- menu

  <details><summary>View table</summary>
  <p>
  
    |   product_id  | product_name  |   price   |
    |---------------|---------------|-----------|
    |       1       |     sushi     |   10      |
    |       2       |     curry     |   15      |
    |       3       |     ramen     |   12      |
  
  </p>
  </details>

- members

  <details><summary>View table</summary>
  <p>
  
    |  customer_id  |    join_date      |
    |---------------|-------------------|
    |       A       |    2021-01-07     |
    |       B       |    2021-01-09     |
  
  </p>
  </details>



# ✒️ Case Study Questions
Each of the following case study questions can be answered using a single SQL statement:

1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

## Bonus Questions
> Danny also seeks to understand the ranking of customer products, with a specific requirement: he only wants to rank products purchased by loyalty program members. For non-member purchases, he expects the ranking values to be null.

In other words, Danny wants to:

- Rank products based on purchase frequency and revenue generated
- Exclude non-member purchases from the ranking
- Display null ranking values for non-member purchases

This will enable Danny to gain insights into the purchasing behavior of his loyalty program members and tailor his marketing efforts accordingly


# 🏆 Solutions
  <details><summary>View solution</summary>
  <p>
  
## 1. Total amount spent by each customer

```sql
    --Using Joins
        SELECT s.customer_id, SUM(m.price) AS [Amount Spent]
        FROM sales s
        LEFT JOIN menu m
        ON m.product_id = s.product_id
        GROUP BY s.customer_id

    --Alternatively - Using Correlated Subquery
        SELECT  DISTINCT  s1.customer_id, 
                (SELECT SUM(m.price)
                FROM menu m, sales s2
                WHERE s1.customer_id = s2.customer_id AND s2.product_id = m.product_id) AS [Amount Spent]
        FROM sales s1

    --Alternatively - Using Window Functions
        SELECT DISTINCT s.customer_id,
                SUM(m.price) OVER(PARTITION BY s.customer_id) AS [Amount Spent]
        FROM sales s
        LEFT JOIN menu m
        ON m.product_id = s.product_id
        ORDER BY [Amount Spent] DESC
```

## 2. Days each customer visited the restaurant

```sql
    --Using Nested subquery
        SELECT s.customer_id, 
               COUNT(order_date)  AS [Number of Days]
        FROM sales s
        GROUP BY s.customer_id
    --Using correlated subquery
        SELECT DISTINCT  s.customer_id,
                (SELECT COUNT(DISTINCT s1.order_date)
                 FROM sales s1
                 WHERE s.customer_id = s1.customer_id) AS [Number of Days]
        FROM sales s
```

## 3. First item from the menu purchased by each customer

```sql
    --Using Joins
        SELECT DISTINCT s.customer_id, m.product_name
        FROM sales s
        LEFT JOIN menu m
        ON m.product_id = s.product_id
        LEFT JOIN (SELECT DISTINCT customer_id, 
                        MIN(order_date) OVER(PARTITION BY customer_id) AS min_order_date
                    FROM sales) AS sq
        ON s.customer_id = sq.customer_id
        WHERE s.order_date = sq.min_order_date

    --Using correlated subqueries
        SELECT DISTINCT s.customer_id, m.product_name
        FROM sales s
        LEFT JOIN menu m
        ON s.product_id = m.product_id
        WHERE s.order_date = (SELECT MIN(order_date)
                              FROM sales s1
                              WHERE s.customer_id = s1.customer_id)
        ORDER BY s.customer_id, m.product_name
```
## 4. Most purchased item and number of times it was purchased

```sql
    --Using Window Functions
        SELECT sq.product_name,sq.[Number of Orders], RANK() OVER(ORDER BY sq.[Number of Orders] DESC) AS Rank
        FROM
            (SELECT DISTINCT m.product_name, COUNT(s.product_id) OVER(PARTITION BY s.product_id) AS [Number of Orders]
             FROM sales s
             LEFT JOIN menu m
             ON s.product_id = m.product_id) AS sq

    --Using Correlated Subquery
        SELECT *, RANK() OVER(ORDER BY sq.[Number of Orders] DESC) AS Rank  
        FROM    
            (SELECT  m.product_name, 
                    (SELECT COUNT(s.product_id)
                     FROM sales s
                     WHERE m.product_id = s.product_id) AS [Number of Orders]
            FROM menu m) as sq

```

## 5. Most popular item for each customer

```sql
    --Using Window Functions
    
        SELECT *
        FROM    
            (SELECT DISTINCT sq.customer_id, sq.product_name, sq.[Number of Orders], RANK() OVER(PARTITION BY sq.customer_id ORDER BY sq.[Number of Orders] DESC) AS Ranking
            FROM    (SELECT DISTINCT s.customer_id, m.product_name, COUNT(s.product_id) OVER(PARTITION BY s.customer_id, m.product_name) AS [Number of Orders]
                    FROM sales s
                    LEFT JOIN menu m
                    ON s.product_id = m.product_id) AS sq) AS q
        WHERE q.Ranking = 1



    --Using Correlated Subquery
        SELECT DISTINCT s.customer_id, m.product_name, 
                (SELECT COUNT(s1.product_id)
                 FROM sales s1
                 WHERE s.product_id = s1.product_id AND s.customer_id = s1.customer_id) AS [Number of Orders]
        FROM sales s 
        LEFT JOIN menu m
        ON m.product_id = s.product_id
```

## 6. First item purchased after becoming a member

```sql
    --Using correlated subquery method
        SELECT DISTINCT sq.customer_id, sq.date_diff, m.product_name
        FROM
            (SELECT s.customer_id, DATEDIFF("DAY",me.join_date,s.order_date) AS date_diff, s.product_id
            FROM members me
            LEFT JOIN sales s
            ON me.customer_id = s.customer_id
            WHERE DATEDIFF("DAY",me.join_date,s.order_date) >= 0) AS sq
        LEFT JOIN menu m
        ON m.product_id = sq.product_id
        WHERE sq.date_diff = (SELECT DISTINCT MIN(DATEDIFF("DAY",m1.join_date,s1.order_date)) OVER(PARTITION BY s1.customer_id)
                              FROM members m1
                              LEFT JOIN sales s1
                              ON m1.customer_id =  s1.customer_id
                              WHERE DATEDIFF("DAY",m1.join_date,s1.order_date) >= 0 AND s1.customer_id = sq.customer_id )

    --Alternatively
            SELECT s.customer_id, s.product_id, m.product_name, s.order_date, DATEDIFF("DAY",me.join_date,s.order_date)
            FROM members me
            LEFT JOIN sales s
            ON me.customer_id = s.customer_id
            LEFT JOIN menu m
            ON s.product_id = m.product_id
            WHERE DATEDIFF("DAY",me.join_date,s.order_date) >= 0 AND  DATEDIFF("DAY",me.join_date,s.order_date) = (SELECT DISTINCT MIN(DATEDIFF("DAY",m1.join_date,s1.order_date)) OVER(PARTITION BY s1.customer_id)
                                                                                                                   FROM members m1
                                                                                                                   LEFT JOIN sales s1
                                                                                                                   ON m1.customer_id =  s1.customer_id
                                                                                                                   WHERE DATEDIFF("DAY",m1.join_date,s1.order_date) >= 0 AND s1.customer_id = s.customer_id )
    --Alternatively
            SELECT *
            FROM
                (SELECT s.customer_id, s.product_id, m.product_name, s.order_date, RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS List
                FROM members me
                LEFT JOIN sales s
                ON me.customer_id = s.customer_id
                LEFT JOIN menu m
                ON s.product_id = m.product_id
                WHERE s.order_date >= me.join_date) AS sq
            WHERE sq.List = 1
```

## 7. Last item purchased just before becoming a member

```sql
    -- One Method
        SELECT DISTINCT sq.customer_id, m.product_name
        FROM
            (SELECT s.customer_id, DATEDIFF("DAY",me.join_date,s.order_date) AS date_diff, s.product_id
            FROM members me
            LEFT JOIN sales s
            ON me.customer_id = s.customer_id
            WHERE DATEDIFF("DAY",me.join_date,s.order_date) < 0) AS sq
        LEFT JOIN menu m
        ON m.product_id = sq.product_id
        WHERE sq.date_diff = (SELECT DISTINCT MAX(DATEDIFF("DAY",m1.join_date,s1.order_date)) OVER(PARTITION BY s1.customer_id)
                              FROM members m1
                              LEFT JOIN sales s1
                              ON m1.customer_id =  s1.customer_id
                              WHERE DATEDIFF("DAY",m1.join_date,s1.order_date) < 0 AND s1.customer_id = sq.customer_id )

    -- Alternatively:
        SELECT me.customer_id, mn.product_name, DATEDIFF("DAY",me.join_date,s.order_date)
        FROM members me
        LEFT JOIN sales s
        ON s.customer_id = me.customer_id
        LEFT JOIN menu mn
        ON mn.product_id = s.product_id
        WHERE DATEDIFF("DAY",me.join_date,s.order_date) = (SELECT DISTINCT MAX(DATEDIFF("DAY",m1.join_date,s1.order_date)) OVER(PARTITION BY s1.customer_id)
                                                          FROM members m1
                                                          LEFT JOIN sales s1
                                                          ON m1.customer_id =  s1.customer_id
                                                          WHERE DATEDIFF("DAY",m1.join_date,s1.order_date) < 0 AND s1.customer_id = s.customer_id )

    --Alternatively
            SELECT *
            FROM
                (SELECT s.customer_id, s.product_id, m.product_name, s.order_date, RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS List
                FROM members me
                LEFT JOIN sales s
                ON me.customer_id = s.customer_id
                LEFT JOIN menu m
                ON s.product_id = m.product_id
                WHERE s.order_date < me.join_date) AS sq
            WHERE sq.List = 1
```

# Q8. Total items and amount spent for each member before becoming a member

```sql
    --Using Joins and Window Functions
        SELECT DISTINCT s.customer_id, m.product_name, 
               SUM(m.price) OVER(PARTITION BY m.product_name, s.customer_id) AS [Total Cost]
        FROM members me
        LEFT JOIN sales s
        ON me.customer_id = s.customer_id
        LEFT JOIN menu m
        ON s.product_id = m.product_id
        WHERE DATEDIFF("DAY",me.join_date,s.order_date) < 0

    --Using Joins and Group BY
        SELECT DISTINCT s.customer_id, m.product_name, SUM(m.price) AS [Total Cost]
        FROM members me
        LEFT JOIN sales s
        ON me.customer_id = s.customer_id
        LEFT JOIN menu m
        ON s.product_id = m.product_id
        WHERE me.join_date > s.order_date
        GROUP BY m.product_name, s.customer_id

    --Using Correlated subqueries
        SELECT DISTINCT me.customer_id, mn.product_name ,(SELECT SUM(m.price) 
                                                          FROM sales s
                                                          LEFT JOIN menu m
                                                          ON s.product_id = m.product_id
                                                          WHERE me.customer_id = s.customer_id AND mn.product_name = m.product_name
                                                          AND DATEDIFF("DAY",me.join_date,s.order_date) < 0)
        FROM  members me
        LEFT JOIN sales s1
        ON me.customer_id = s1.customer_id
        LEFT JOIN menu mn
        ON mn.product_id = s1.product_id
```

## 9. $1 = 10 points and sushi = 2x multiplier calculate each members points

```sql
    --Using Case statements
        SELECT sq.customer_id, SUM(sq.price * sq.multiplier * 10) AS Points  
        FROM
            (SELECT s.customer_id, mn.product_name, mn.price,
                    CASE 
                        WHEN [product_name] = 'sushi' THEN 2   
                        ELSE 1
                    END AS multiplier
            FROM sales s
            LEFT JOIN menu mn
            ON s.product_id = mn.product_id) AS sq
        GROUP BY sq.customer_id
```

## 10. 2x on all items in the first week of joining

```sql
    --Using case statements
        SELECT sq.customer_id, SUM(sq.price * sq.[membership multiplier] * 10) AS Points  
        FROM
            (SELECT s.customer_id, mn.product_name, mn.price, s.order_date,
                    CASE 
                        WHEN [product_name] = 'sushi'AND DATENAME("month",s.order_date) = 'January'  THEN 2   -- use single quotes with case statements
                        ELSE 1
                    END AS multiplier,
                    CASE 
                        WHEN s.order_date BETWEEN me.join_date AND DATEADD("Week",1,me.join_date) THEN 2
                        ELSE 1
                    END AS [membership multiplier]
            FROM sales s
            LEFT JOIN menu mn
            ON s.product_id = mn.product_id
            LEFT JOIN members me
            ON me.customer_id = s.customer_id
            ) AS sq
        GROUP BY sq.customer_id
```

- Bonus Questions

```sql
    -- CREATE VIEW [Total] AS
    SELECT s.customer_id, s.order_date, mn.product_name, mn.price, 
            CASE
                WHEN s.order_date >= me.join_date THEN 'Y'
                ELSE 'N'
            END AS [member]
    -- INTO FullTable
    FROM sales s
    LEFT JOIN menu mn
    ON mn.product_id = s.product_id
    LEFT JOIN members me
    ON me.customer_id = s.customer_id;
```

  
  </p>
  </details>

