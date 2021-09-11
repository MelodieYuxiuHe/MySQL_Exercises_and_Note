## Hard

### 262. Trips and Users [Table Reshape] [Group By]

```mysql
# Write your MySQL query statement below

WITH ALL_PEOPLE AS (SELECT Id, Client_Id AS 'People', City_Id, Status, Request_at FROM Trips
UNION ALL 
SELECT Id, Driver_Id AS 'People', City_Id, Status, Request_at FROM Trips),

UNBANNED_PEOPLE_WITH_STATUS AS (SELECT * FROM ALL_PEOPLE 
                        LEFT JOIN Users u ON u.Users_Id = ALL_PEOPLE.People
                                GROUP BY Id
                                Having Banned = "No"),

TMP AS (SELECT Request_at, CASE WHEN Status = 'completed' THEN 0
ELSE 1 END as Status_Adj, Id FROM UNBANNED_PEOPLE_WITH_STATUS)

SELECT Request_at AS 'Day', round(Sum(Status_Adj)/Count(Status_Adj),2) AS 'Cancellation Rate'
FROM TMP 

WHERE Request_at between '2013-10-01' AND '2013-10-03'
GROUP BY Request_at
_____________________________________________________________

```

```mysql
#Others Solution
Select request_at Day, 
    ROUND(SUM(if(status like "cancelled_by_%",1,0)) / COUNT(status),2) `Cancellation Rate`
from trips t 
JOIN users u1 ON t.client_id = u1.users_id AND u1.banned = "No"
JOIN users u2 ON t.driver_id = u2.users_id AND u2.banned = "No"
where request_at between '2013-10-01' and '2013-10-03'
GROUP BY request_at
```



### 185. Department Top Three Salaries [Dense Rank]

```mysql
DENSE_RANK() OVER (PARTITION BY fiscal_year ORDER BY Salary DESC) sales_rank
```

https://www.mysqltutorial.org/mysql-window-functions/mysql-dense_rank-function/

```mysql
WITH TMP AS (
SELECT e.Name as 'Employee_Name', e.Salary, d.Id as 'Depart_Id', d.Name AS 'Depart_Name' FROM Employee e JOIN Department d ON e.DepartmentId = d.Id),


TMP2 AS (SELECT Employee_Name, Salary, Depart_Id, Depart_Name, DENSE_RANK() OVER (PARTITION BY Depart_Id ORDER BY Salary DESC) salary_Rank
FROM TMP)

SELECT Depart_Name AS 'Department', Employee_Name AS 'Employee', Salary from tmp2
WHERE salary_Rank <= 3
```

```mysql
#Others solution
select d.name as department, e.employee, e.salary from
(select id, name as employee, salary, departmentid, dense_rank() over (partition by departmentid order by salary desc) as 'rank' from employee) e
join Department d
on d.id = e.departmentid
where e.rank <= 3
```

### 1917. Leetcodify Friends Recommendations [Join] [Left Join] [Null]

```mysql

WITH TMP AS 
(SELECT * FROM (SELECT a.user_id AS 'a_user', b.user_id AS 'b_user', a.song_id, a.day, count(distinct(a.song_id)) AS 'unique_song_count' FROM listens a
JOIN listens b ON
 a.song_id = b.song_id 
 AND a.day = b.day
 AND a.user_id <> b.user_id
 # AND a.user_id < b.user_id
 GROUP BY a.day, a.user_id, b.user_id) x
 WHERE unique_song_count >= 3)
 
 
 SELECT DISTINCT(TMP.a_user) AS user_id, TMP.b_user AS 'recommended_id'
 FROM TMP 
 LEFT JOIN friendship fa ON 
 TMP.a_user = fa.user1_id
 AND TMP.b_user = fa.user2_id
 LEFT JOIN friendship fb ON 
 TMP.a_user = fb.user2_id 
 AND TMP.b_user = fb.user1_id
 WHERE (fa.user2_id IS NULL)
 AND (fb.user1_id IS NULL)

```

### 141. Find the Quiet Students in All Exams [Rank] [Filter use 2 Arrays]

```sql
#Find out what's the highest rank (using rank to allow the tie situations)
WITH TMP_HIGH AS (SELECT * FROM (SELECT exam_id, student_id AS 'High_Student', rank() over (partition by exam_id order by score desc) AS 'HIGH_RANK'
FROM Exam) t1 WHERE HIGH_RANK = 1),

#Find out what's the lowest rank (using rank to allow the tie situations)
TMP_LOW AS (SELECT * FROM (SELECT exam_id, student_id AS 'Low_Student', rank() over (partition by exam_id order by score ASC) AS 'LOW_RANK'
FROM Exam) t2 WHERE LOW_RANK = 1),

#Students that ranked high
STUDENT_HIGH AS (
    SELECT student_id FROM student JOIN TMP_HIGH ON TMP_HIGH.High_Student=student.student_id),
 
#Students that ranked low 
STUDENT_LOW AS (
    SELECT student_id FROM student JOIN TMP_LOW ON TMP_LOW.Low_Student=student.student_id),
   
#All students that ranked high or low
ALL_STUDENT AS ((SELECT * FROM STUDENT_HIGH) UNION (SELECT * FROM STUDENT_LOW))
    
# Find out student attended the exam AND not in the list above    
SELECT DISTINCT(student.student_id), student.student_name 
FROM Exam
LEFT JOIN Student ON 
Student.student_id = Exam.student_id
WHERE Student.student_id NOT IN (SELECT * FROM ALL_STUDENT)
ORDER BY student_id

```

### 601. Human Traffic of Stadium[Consecutive Number]

```sql
# Use row_number() to find out 'breaks' in id, and group by them
WITH GROUP_ID AS (
             SELECT id,
             id - ROW_NUMBER() over (order by id) group_id,
             people, 
             visit_date
             FROM stadium
             WHERE people >= 100),
# at least 3 consecutive number 
CONSEC AS (SELECT id, visit_date, people, group_id 
          FROM GROUP_ID
          GROUP BY group_id
          HAVING COUNT(group_id) >= 3)
                    
SELECT id, visit_date, people FROM GROUP_ID
WHERE group_id IN (SELECT group_id FROM CONSEC)
ORDER BY visit_date
```

### 602. Friend Requests II: Who Has the Most Friends [Pairwise] [Union]

```sql
#The key to deal with pairewise questions (A:B, B:A) is to do an Union between table 1 and swap-column table 2. REMEMBER TO CLEARLY SPECIFY WHICH COLUMNS TO UNION, CHANGING THE NAME WON'T MAKE THE TRICK
WITH ALL_REQUEST AS
(SELECT requester_id, accepter_id FROM request_accepted a
UNION 
SELECT accepter_id as requester_id, requester_id as accepter_id
FROM request_accepted b)
                  
SELECT requester_id as id, COUNT(DISTINCT accepter_id) AS num
FROM ALL_REQUEST
GROUP BY requester_id
HAVING COUNT(DISTINCT(accepter_id)) >= 
(SELECT COUNT(DISTINCT(accepter_id))
 FROM ALL_REQUEST 
 GROUP BY requester_id 
 ORDER BY COUNT(DISTINCT(accepter_id)) DESC 
 LIMIT 1)
```



### 1635. Hopper Company Queries I [Join on Unequal Condition] [Create a reference date table]

```sql
# Create a month and year table
WITH recursive MONTH_DATE AS (
    SELECT 
    2020 as yeart,
    1 as montht
    UNION ALL
    SELECT yeart, montht+1 as montht
    FROM MONTH_DATE
    WHERE montht<12
),

# Creat a table with the last day in each month for 2020 (prepare for the active driver table)
MONTH_DATE2 AS (SELECT montht, yeart, lastday,
                STR_TO_DATE(DATE_TMP, "%d,%m,%Y") AS DATE_TMP 
                FROM (SELECT montht, yeart, lastday, CONCAT(lastday, ",", montht, ",", yeart) AS DATE_TMP FROM (
SELECT 
montht, 
yeart, 
CASE WHEN montht in (1,3,5,7,8,10,12) THEN 31
    WHEN montht in (4,6,9,11) THEN 30
    WHEN montht = 2 THEN 29
    END AS lastday
FROM MONTH_DATE
)t) t1),

# LEFT JOIN ON 'UNEQUAL CONDITION'
active_driver AS (SELECT montht, COUNT(DISTINCT(driver_id)) AS 'active_drivers'
FROM  MONTH_DATE2
LEFT JOIN Drivers
ON MONTH_DATE2.DATE_TMP >= Drivers.join_date
GROUP BY MONTH_DATE2.montht),

# ACTIVE DRIVER
active_ride AS (
SELECT EXTRACT(MONTH FROM Rides.requested_at) AS 'ride_month', count(Rides.ride_id) AS 'accepted_rides' FROM AcceptedRides
LEFT JOIN Rides ON AcceptedRides.ride_id = Rides.ride_id
WHERE EXTRACT(YEAR FROM Rides.requested_at) = 2020
GROUP BY ride_month)

# Combine them together
SELECT active_driver.montht AS  `month`, active_drivers, ifnull(accepted_rides, 0) as accepted_rides FROM 
MONTH_DATE2 LEFT JOIN active_driver
ON active_driver.montht = MONTH_DATE2.montht
LEFT JOIN active_ride
ON active_driver.montht = active_ride.ride_month
```

### 1645. Hopper Company Queries II [ifnull]

```sql
# Create a month and year table
WITH recursive MONTH_DATE AS (
    SELECT 
    2020 as yeart,
    1 as montht
    UNION ALL
    SELECT yeart, montht+1 as montht
    FROM MONTH_DATE
    WHERE montht<12
),

# Creat a table with the last day in each month for 2020 (prepare for the active driver table)
MONTH_DATE2 AS (SELECT montht, yeart, lastday,
                STR_TO_DATE(DATE_TMP, "%d,%m,%Y") AS DATE_TMP 
                FROM (SELECT montht, yeart, lastday, CONCAT(lastday, ",", montht, ",", yeart) AS DATE_TMP FROM (
SELECT 
montht, 
yeart, 
CASE WHEN montht in (1,3,5,7,8,10,12) THEN 31
    WHEN montht in (4,6,9,11) THEN 30
    WHEN montht = 2 THEN 29
    END AS lastday
FROM MONTH_DATE
)t) t1),


active_driver AS (SELECT montht, COUNT(driver_id) AS 'active_drivers'
FROM  MONTH_DATE2
LEFT JOIN Drivers
ON MONTH_DATE2.DATE_TMP >= Drivers.join_date
GROUP BY montht),

# ACTIVE DRIVER
active_ride AS (
SELECT EXTRACT(MONTH FROM Rides.requested_at) AS 'ride_month', 
ifnull(COUNT(DISTINCT(driver_id)), 0) AS 'Number_Drivers'
FROM AcceptedRides
LEFT JOIN Rides ON AcceptedRides.ride_id = Rides.ride_id
WHERE EXTRACT(YEAR FROM Rides.requested_at) = 2020
GROUP BY ride_month)

# Combine them together
SELECT active_driver.montht AS `month`,
ifnull(round(ifnull(Number_Drivers, 0)/active_drivers,4)*100, 0) AS 'working_percentage' FROM 
MONTH_DATE2 LEFT JOIN active_driver
ON active_driver.montht = MONTH_DATE2.montht
LEFT JOIN active_ride
ON active_driver.montht = active_ride.ride_month

```

### 1097. Game Play Analysis V

```sql
# When is the install date, and day1 for each player
WITH PLAYER_FIRST_DATE AS (SELECT player_id, min(event_date) as install_date, date_add(min(event_date), interval +1 day) as day1_date
FROM Activity GROUP BY player_id),

# Group by date, and count distinct player to get number of install in each day
NumberInstalled as (SELECT COUNT(DISTINCT(player_id)) as installnum, install_date FROM PLAYER_FIRST_DATE
GROUP BY install_date),

# Join the activity table using the 'day1' (install date +1) and count how many users returned, groun by 'INSTALL DATE', 'install date' is a key to connect the NumberInstalled and PLAYER_RETURNED table. The analysis is based on install date.

PLAYER_RETURNED AS (SELECT COUNT(DISTINCT(a.player_id)) as 'returnednum', pfd.install_date
FROM PLAYER_FIRST_DATE pfd JOIN
Activity a ON pfd.day1_date = a.event_date
AND pfd.player_id = a.player_id
GROUP BY pfd.install_date)

# When do left join, be careful with group by level: if group by at PLAYER_RETURNED.install_date, days with user retention rate to be 0 will not be returned 
SELECT NumberInstalled.install_date as 'install_dt',
NumberInstalled.installnum as 'installs',
round(ifnull(returnednum/installnum,0),2) as 'Day1_retention'
FROM NumberInstalled
LEFT JOIN PLAYER_RETURNED
ON NumberInstalled.install_date = PLAYER_RETURNED.install_date
GROUP BY NumberInstalled.install_date
 
```









### 1919 Leetcodify Similar Friends [Group By] [Reshape] [Join] [Count]

```mysql
WITH TMP AS (SELECT user_id, day, song_id, CONCAT(day,"-", song_id) AS UNIQUE_Key FROM Listens GROUP BY user_id, song_id, day),

TMP2 AS (SELECT a.day, a.song_id, a.UNIQUE_Key, a.user_id AS 'A_USER', b.user_id AS 'B_USER' FROM TMP a
JOIN TMP b
ON a.UNIQUE_Key = b.UNIQUE_Key
where a.user_id < b.user_id),

TMP3 AS (SELECT * FROM TMP2 JOIN Friendship f ON f.user1_id = TMP2.A_USER AND user2_id = TMP2.B_USER AND TMP2.A_USER < f.user2_id ),

TMP4 AS (SELECT day, song_id, A_USER, B_USER, COUNT(DISTINCT(song_id)) AS 'Count_Number' FROM TMP3
WHERE A_USER < B_USER
GROUP BY day, A_USER, B_USER)

SELECT DISTINCT A_USER AS 'user1_id', B_USER AS 'user2_id' FROM TMP4
WHERE Count_Number >= 3
GROUP BY A_USER, B_USER
```

```mysql
SELECT DISTINCT  l.user1_id, l.user2_id
FROM
(
    SELECT a.user_id AS user1_id, b.user_id AS user2_id, a.day
    FROM
     listens a
     INNER JOIN
           listens b
           ON a.user_id < b.user_id AND a.song_id = b.song_id AND a.day = b.day
    GROUP BY a.user_id, b.user_id, a.day
    HAVING COUNT(DISTINCT a.song_id) >= 3
) l
 INNER JOIN
    friendship f
    ON l.user1_id = f.user1_id AND l.user2_id = f.user2_id
```







## Easy

### 512. Game Play Analysis 2 [Dense_Rank( )] [Min_Max]

```mysql
with tmp as (select player_id, device_id, DENSE_RANK() OVER (PARTITION BY player_id ORDER BY event_date) as DATE_RANK from activity) 

SELECT tmp.player_id, tmp.device_id from tmp
where DATE_RANK = 1 
```

### 597. Friend Requests I: Overall Acceptance Rate [SELECT DISTINCT A, B] [Row number: COUNT(*)]

```sql
SELECT
ROUND(
    IFNULL(
    (SELECT COUNT(*) FROM (SELECT DISTINCT requester_id, accepter_id FROM RequestAccepted) AS A)
    /
    (SELECT COUNT(*) FROM (SELECT DISTINCT sender_id, send_to_id FROM FriendRequest) AS B),
    0)
, 2) AS accept_rate
```



## Medium

### 534. Game Play Analysis 3 [Cumulative Sum] [Partition By]

```mysql
WITH TMP AS (SELECT player_id, event_date, 
             games_played
             from activity
             group by player_id, event_date)
             
select player_id, event_date as event_date, sum(games_played) over (partition by player_id order by event_date) as games_played_so_far from TMP
```

Cumulative Sum using Sum Over Partition By

https://popsql.com/learn-sql/mysql/how-to-calculate-cumulative-sum-running-total-in-mysql

### 550. Game Play Analysis IV [JOIN]

```sql
#Total number of players
WITH NumberofPlayer AS 
(SELECT COUNT(DISTINCT(player_id)) as 'UserBase' FROM Activity) 
             
SELECT round(ifnull(count(distinct(t1.player_id))/(SELECT UserBase FROM NumberofPlayer), 0),2) as 'fraction' FROM

#Calculate day 1 (install date +1) and join the activity table and count to get total number of returned players

((SELECT player_id, min(event_date) as install_day, date_add(min(event_date), interval +1 day) as day1
FROM Activity
GROUP BY player_id) t1
JOIN
Activity a
ON a.event_date = t1.day1
# Remember to also join on player id!
AND a.player_id = t1.player_id)
```



### 1045. Customers Who Bought All Products [Group By] [SELECT DISTINCT]

```sql
SELECT DISTINCT customer_id FROM
Customer c
LEFT JOIN Product p
ON c.product_key = p.product_key
GROUP BY customer_id
HAVING COUNT(DISTINCT(c.product_key)) = (SELECT COUNT(DISTINCT(product_key)) FROM Product)
```

