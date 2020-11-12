
MEDIUM

1098. Unpopular Books

```MY SQL
SELECT B.book_id, B.name
FROM Books B LEFT JOIN Orders O
ON B.book_id = O.book_id
WHERE DATEDIFF('2019-05-23', B.available_from) >= 30
GROUP BY B.book_id
HAVING SUM(IF(DATEDIFF('2019-06-23', O.dispatch_date) <= 365, O.quantity, 0)) < 10
```
Note: 
1) The last line 'SUM(IF' is used because some books did not have order last year. In the Orders table, there is no record for last year (no XXX, 2019-07-23, 0). Those books satisfy the condition that 'quantity less than 10 last year'. So we will create conditional sum and assign 0 to these books.
2) We use LEFT JOIN instead join is that we focus on books. The reason is similar. It is possible that we have a book that wasn't sold even one. (No corresponding orders). But this book satisfies the condition. If we use JOIN instead of LEFT JOIN, we may lose these books.
3) Consider DATEDIFF() when need to deal with date range

1067. Sellers With No Sales

My answer:

```MY SQL
SELECT seller_name
FROM Seller S 
WHERE S.seller_id NOT IN (SELECT seller_id FROM Orders WHERE YEAR(Orders.sale_date) = 2020)
ORDER BY seller_name
```

Answer in Discussion Session (https://leetcode.com/problems/sellers-with-no-sales/discuss/895961/No-subquery)

```MY SQL
select s.seller_name
  from Seller s
  left join Orders o
    on s.seller_id = o.seller_id and year(sale_date) = 2020 
 where o.seller_id is null
 order by s.seller_name
```
Note:
1) It only left joined (interested in sellers and consider situation that seller may have zero order recorded in Orders table) on the 2020 year orders. The result seller_id (from Orders) will be null for all the sellers that have non-2020 orders or not sold anything (no info in Orders table). If the seller_id in result is not null, it means it matched with an order in 2020. 
2) What if a seller has two orders in 2019 and 2020? The 2020 order will be matched and that seller_id will not be null. We won't select this seller_id using this method.
3) Left/Right Join, O.seller_id is not the same as S.seller_id


1596. The Most Frequently Ordered Products for Each Customer
```MY SQL
WITH t1 AS (SELECT customer_id, product_id, RANK() OVER (PARTITION BY customer_id ORDER BY COUNT(order_date) DESC) AS rk FROM Orders GROUP BY customer_id, product_id)

SELECT t1.customer_id, t1.product_id, P.product_name
FROM t1, Products P
WHERE rk = 1
AND t1.product_id = P.product_id
ORDER BY t1.customer_id, t1.product_id 
```
Note:
The question asked for 'most frequent product'. Thus we count date.
Partition by customer_id, we rank: for each customer, how many different order for one product?

1613. Find the Missing IDs

```MY SQL
WITH RECURSIVE cte AS (
    SELECT 1 AS counter  -- anchor member
    UNION ALL
    SELECT counter + 1 FROM cte WHERE counter < (SELECT MAX(customer_id) FROM customers) -- recursive member that references to the CTE name
)
SELECT counter AS ids FROM cte
WHERE counter NOT IN (SELECT customer_id FROM customers)
```
Note: 
1) Recursive CTE: https://www.mysqltutorial.org/mysql-recursive-cte/
2) cte selects all the number from 1 to the max customer id, then we exclude whatever already exist

EASY

1633. Percentage of Users Attended a Contest
My Answer:
```MY SQL
SELECT contest_id, ROUND(COUNT(R.user_id)/(SELECT COUNT(DISTINCT(user_id)) FROM Users)*100,2) AS percentage
FROM Register R
GROUP BY contest_id
ORDER BY percentage DESC, contest_id
```

196. Delete Duplicate Emails

My Answer:

```MY SQL
WITH tmp AS (SELECT MIN(Id) FROM Person GROUP BY Email)

DELETE FROM Person
WHERE Id NOT IN (SELECT * FROM tmp)
```
Note:
Use 'DELETE' here

197. Rising Temperature

```MY SQL
SELECT x.id
FROM Weather x JOIN Weather y
ON DATEDIFF(x.recordDate, y.recordDate) = 1
AND x.Temperature > y.Temperature
```

```MY SQL
SELECT w.id FROM Weather w JOIN Weather p
ON w.recordDate = DATE_ADD(p.recordDate, INTERVAL 1 DAY) AND w.Temperature > p.Temperature
```
Note:
1) DATE_ADD function and DATEDIFF function
2) JOIN ON, specify conditions for join




