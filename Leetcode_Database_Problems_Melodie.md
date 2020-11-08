
Medium

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