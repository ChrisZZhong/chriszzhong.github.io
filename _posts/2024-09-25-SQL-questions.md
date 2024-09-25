---
layout: post
title: "SQL"
date: 2023-01-18
description: "SQL leetcode questions"
tag: SQL
---

# SQL questions

## Search

## [🟢 1821. Find Customers With Positive Revenue this Year ](https://leetcode.cn/problems/find-customers-with-positive-revenue-this-year/description/) 

**Description**
<div class="elfjS" data-track-load="description_content"><p>Table: <code>Customers</code></p>

<pre>+--------------+------+
| Column Name  | Type |
+--------------+------+
| customer_id  | int  |
| year         | int  |
| revenue      | int  |
+--------------+------+
(customer_id, year) is the primary key (combination of columns with unique values) for this table.
This table contains the customer ID and the revenue of customers in different years.
Note that this revenue can be negative.
</pre>

<p>Write a solution to report the customers with <strong>postive revenue</strong> in the year 2021.</p>

<p>Return the result table in <strong>any order</strong>.</p>

<p>The&nbsp;result format is in the following example.</p>

<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Customers table:
+-------------+------+---------+
| customer_id | year | revenue |
+-------------+------+---------+
| 1           | 2018 | 50      |
| 1           | 2021 | 30      |
| 1           | 2020 | 70      |
| 2           | 2021 | -50     |
| 3           | 2018 | 10      |
| 3           | 2016 | 50      |
| 4           | 2021 | 20      |
+-------------+------+---------+
<strong>Output:</strong> 
+-------------+
| customer_id |
+-------------+
| 1           |
| 4           |
+-------------+
<strong>Explanation:</strong> 
Customer 1 has revenue equal to 30 in the year 2021.
Customer 2 has revenue equal to -50 in the year 2021.
Customer 3 has no revenue in the year 2021.
Customer 4 has revenue equal to 20 in the year 2021.
Thus only customers 1 and 4 have positive revenue in the year 2021.
</pre>
</div>

**SQL**

```mysql
select customer_id from customers where revenue > 0 and year = 2021
```

## [🟢 183. Customers Who Never Order](https://leetcode.cn/problems/customers-who-never-order/)

**Description**
<div class="elfjS" data-track-load="description_content"><p>Table: <code>Customers</code></p>

<pre>+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
+-------------+---------+
id is the primary key (column with unique values) for this table.
Each row of this table indicates the ID and name of a customer.
</pre>

<p>Table: <code>Orders</code></p>

<pre>+-------------+------+
| Column Name | Type |
+-------------+------+
| id          | int  |
| customerId  | int  |
+-------------+------+
id is the primary key (column with unique values) for this table.
customerId is a foreign key (reference columns) of the ID from the Customers table.
Each row of this table indicates the ID of an order and the ID of the customer who ordered it.
</pre>

<p>Write a solution to find all customers who never order anything.</p>

<p>Return the result table in <strong>any order</strong>.</p>

<p>The result format is in the following example.</p>

<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Customers table:
+----+-------+
| id | name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
Orders table:
+----+------------+
| id | customerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
<strong>Output:</strong> 
+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+
</pre>
</div>

**SQL**

```mysql
select name as Customers
from customers
where id not in (
    select customerId from orders
)
```

## [🟢 1873. Calculate Special Bonus](https://leetcode.cn/problems/calculate-special-bonus/description/)

**Description**
<div class="elfjS" data-track-load="description_content"><p>Table: <code>Employees</code></p>

<pre>+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| employee_id | int     |
| name        | varchar |
| salary      | int     |
+-------------+---------+
employee_id is the primary key (column with unique values) for this table.
Each row of this table indicates the employee ID, employee name, and salary.
</pre>

<p>&nbsp;</p>

<p>Write a solution to calculate the bonus of each employee. The bonus of an employee is <code>100%</code> of their salary if the ID of the employee is <strong>an odd number</strong> and <strong>the employee's name does not start with the character </strong><code>'M'</code>. The bonus of an employee is <code>0</code> otherwise.</p>

<p>Return the result table ordered by <code>employee_id</code>.</p>

<p>The&nbsp;result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Employees table:
+-------------+---------+--------+
| employee_id | name    | salary |
+-------------+---------+--------+
| 2           | Meir    | 3000   |
| 3           | Michael | 3800   |
| 7           | Addilyn | 7400   |
| 8           | Juan    | 6100   |
| 9           | Kannon  | 7700   |
+-------------+---------+--------+
<strong>Output:</strong> 
+-------------+-------+
| employee_id | bonus |
+-------------+-------+
| 2           | 0     |
| 3           | 0     |
| 7           | 7400  |
| 8           | 0     |
| 9           | 7700  |
+-------------+-------+
<strong>Explanation:</strong> 
The employees with IDs 2 and 8 get 0 bonus because they have an even employee_id.
The employee with ID 3 gets 0 bonus because their name starts with 'M'.
The rest of the employees get a 100% bonus.
</pre>
</div>

**SQL**

```mysql
select employee_id,
    case
        when name not like "m%" and employee_id % 2 != 0 then salary
        else 0 
        end as bonus
from Employees 
order by employee_id asc
```

## [1398. Customers Who Bought Products A and B but Not C](https://leetcode.cn/problems/customers-who-bought-products-a-and-b-but-not-c/description/)

**Description**

<div class="elfjS" data-track-load="description_content"><p>Table: <code>Customers</code></p>

<pre>+---------------------+---------+
| Column Name         | Type    |
+---------------------+---------+
| customer_id         | int     |
| customer_name       | varchar |
+---------------------+---------+
customer_id is the column with unique values for this table.
customer_name is the name of the customer.</pre>

<p>&nbsp;</p>

<p>Table: <code>Orders</code></p>

<pre>+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| order_id      | int     |
| customer_id   | int     |
| product_name  | varchar |
+---------------+---------+
order_id is the column with unique values for this table.
customer_id is the id of the customer who bought the product "product_name".
</pre>

<p>&nbsp;</p>

<p>Write a solution&nbsp;to report the customer_id and customer_name of customers who bought products <strong>"A"</strong>, <strong>"B"</strong> but did not buy the product <strong>"C"</strong> since we want to recommend them to purchase this product.</p>

<p>Return the result table <strong>ordered</strong> by <code>customer_id</code>.</p>

<p>The&nbsp;result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Customers table:
+-------------+---------------+
| customer_id | customer_name |
+-------------+---------------+
| 1           | Daniel        |
| 2           | Diana         |
| 3           | Elizabeth     |
| 4           | Jhon          |
+-------------+---------------+
Orders table:
+------------+--------------+---------------+
| order_id   | customer_id  | product_name  |
+------------+--------------+---------------+
| 10         |     1        |     A         |
| 20         |     1        |     B         |
| 30         |     1        |     D         |
| 40         |     1        |     C         |
| 50         |     2        |     A         |
| 60         |     3        |     A         |
| 70         |     3        |     B         |
| 80         |     3        |     D         |
| 90         |     4        |     C         |
+------------+--------------+---------------+
<strong>Output:</strong> 
+-------------+---------------+
| customer_id | customer_name |
+-------------+---------------+
| 3           | Elizabeth     |
+-------------+---------------+
<strong>Explanation:</strong> Only the customer_id with id 3 bought the product A and B but not the product C.
</pre>
</div>

**SQL**

```mysql
select c.customer_id, c.customer_name 
from Orders o
join Customers c
on c.customer_id = o.customer_id
group by customer_id
having 
    sum(product_name = "A") > 0 and
    sum(product_name = "B") > 0 and
    sum(product_name = "C") = 0 
```


## [1112. Highest Grade For Each Student](https://leetcode.cn/problems/highest-grade-for-each-student/description/)

Description:

<div class="elfjS" data-track-load="description_content"><p>Table: <code>Enrollments</code></p>

<pre>+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| student_id    | int     |
| course_id     | int     |
| grade         | int     |
+---------------+---------+
(student_id, course_id) is the primary key (combination of columns with unique values) of this table.
grade is never NULL.
</pre>

<p>&nbsp;</p>

<p>Write a solution to find the highest grade with its corresponding course for each student. In case of a tie, you should find the course with the smallest <code>course_id</code>.</p>

<p>Return the result table ordered by <code>student_id</code> in <strong>ascending order</strong>.</p>

<p>The&nbsp;result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Enrollments table:
+------------+-------------------+
| student_id | course_id | grade |
+------------+-----------+-------+
| 2          | 2         | 95    |
| 2          | 3         | 95    |
| 1          | 1         | 90    |
| 1          | 2         | 99    |
| 3          | 1         | 80    |
| 3          | 2         | 75    |
| 3          | 3         | 82    |
+------------+-----------+-------+
<strong>Output:</strong> 
+------------+-------------------+
| student_id | course_id | grade |
+------------+-----------+-------+
| 1          | 2         | 99    |
| 2          | 2         | 95    |
| 3          | 3         | 82    |
+------------+-----------+-------+
</pre>
</div>

SQL

```mysql
select student_id, course_id, grade
from (
    select *, 
        rank() over (partition by student_id order by grade desc, course_id) as rnk
    from Enrollments
) ranked_enrollments
where rnk = 1;
```


## [1407. Top Travellers](https://leetcode.cn/problems/top-travellers/description/)

Description:
<div class="elfjS" data-track-load="description_content"><p>Table: <code>Users</code></p>

<pre>+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| name          | varchar |
+---------------+---------+
id is the column with unique values for this table.
name is the name of the user.
</pre>

<p>&nbsp;</p>

<p>Table: <code>Rides</code></p>

<pre>+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| user_id       | int     |
| distance      | int     |
+---------------+---------+
id is the column with unique values for this table.
user_id is the id of the user who traveled the distance "distance".
</pre>

<p>&nbsp;</p>

<p>Write a solution&nbsp;to report the distance traveled by each user.</p>

<p>Return the result table ordered by <code>travelled_distance</code> in <strong>descending order</strong>, if two or more users traveled the same distance, order them by their <code>name</code> in <strong>ascending order</strong>.</p>

<p>The&nbsp;result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Users table:
+------+-----------+
| id   | name      |
+------+-----------+
| 1    | Alice     |
| 2    | Bob       |
| 3    | Alex      |
| 4    | Donald    |
| 7    | Lee       |
| 13   | Jonathan  |
| 19   | Elvis     |
+------+-----------+
Rides table:
+------+----------+----------+
| id   | user_id  | distance |
+------+----------+----------+
| 1    | 1        | 120      |
| 2    | 2        | 317      |
| 3    | 3        | 222      |
| 4    | 7        | 100      |
| 5    | 13       | 312      |
| 6    | 19       | 50       |
| 7    | 7        | 120      |
| 8    | 19       | 400      |
| 9    | 7        | 230      |
+------+----------+----------+
<strong>Output:</strong> 
+----------+--------------------+
| name     | travelled_distance |
+----------+--------------------+
| Elvis    | 450                |
| Lee      | 450                |
| Bob      | 317                |
| Jonathan | 312                |
| Alex     | 222                |
| Alice    | 120                |
| Donald   | 0                  |
+----------+--------------------+
<strong>Explanation:</strong> 
Elvis and Lee traveled 450 miles, Elvis is the top traveler as his name is alphabetically smaller than Lee.
Bob, Jonathan, Alex, and Alice have only one ride and we just order them by the total distances of the ride.
Donald did not have any rides, the distance traveled by him is 0.
</pre>
</div>

SQL

```mysql
select u.name, ifnull(sum(distance), 0) as travelled_distance
from rides r
right join users u 
on u.id = r.user_id
group by u.id
order by sum(distance) desc, u.name
```

## [607. Sales Person](https://leetcode.cn/problems/sales-person/description/)

Description:

<div class="elfjS" data-track-load="description_content"><p>Table: <code>SalesPerson</code></p>

<pre>+-----------------+---------+
| Column Name     | Type    |
+-----------------+---------+
| sales_id        | int     |
| name            | varchar |
| salary          | int     |
| commission_rate | int     |
| hire_date       | date    |
+-----------------+---------+
sales_id is the primary key (column with unique values) for this table.
Each row of this table indicates the name and the ID of a salesperson alongside their salary, commission rate, and hire date.
</pre>

<p>&nbsp;</p>

<p>Table: <code>Company</code></p>

<pre>+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| com_id      | int     |
| name        | varchar |
| city        | varchar |
+-------------+---------+
com_id is the primary key (column with unique values) for this table.
Each row of this table indicates the name and the ID of a company and the city in which the company is located.
</pre>

<p>&nbsp;</p>

<p>Table: <code>Orders</code></p>

<pre>+-------------+------+
| Column Name | Type |
+-------------+------+
| order_id    | int  |
| order_date  | date |
| com_id      | int  |
| sales_id    | int  |
| amount      | int  |
+-------------+------+
order_id is the primary key (column with unique values) for this table.
com_id is a foreign key (reference column) to com_id from the Company table.
sales_id is a foreign key (reference column) to sales_id from the SalesPerson table.
Each row of this table contains information about one order. This includes the ID of the company, the ID of the salesperson, the date of the order, and the amount paid.
</pre>

<p>&nbsp;</p>

<p>Write a solution to find the names of all the salespersons who did not have any orders related to the company with the name <strong>"RED"</strong>.</p>

<p>Return the result table in <strong>any order</strong>.</p>

<p>The result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
SalesPerson table:
+----------+------+--------+-----------------+------------+
| sales_id | name | salary | commission_rate | hire_date  |
+----------+------+--------+-----------------+------------+
| 1        | John | 100000 | 6               | 4/1/2006   |
| 2        | Amy  | 12000  | 5               | 5/1/2010   |
| 3        | Mark | 65000  | 12              | 12/25/2008 |
| 4        | Pam  | 25000  | 25              | 1/1/2005   |
| 5        | Alex | 5000   | 10              | 2/3/2007   |
+----------+------+--------+-----------------+------------+
Company table:
+--------+--------+----------+
| com_id | name   | city     |
+--------+--------+----------+
| 1      | RED    | Boston   |
| 2      | ORANGE | New York |
| 3      | YELLOW | Boston   |
| 4      | GREEN  | Austin   |
+--------+--------+----------+
Orders table:
+----------+------------+--------+----------+--------+
| order_id | order_date | com_id | sales_id | amount |
+----------+------------+--------+----------+--------+
| 1        | 1/1/2014   | 3      | 4        | 10000  |
| 2        | 2/1/2014   | 4      | 5        | 5000   |
| 3        | 3/1/2014   | 1      | 1        | 50000  |
| 4        | 4/1/2014   | 1      | 4        | 25000  |
+----------+------------+--------+----------+--------+
<strong>Output:</strong> 
+------+
| name |
+------+
| Amy  |
| Mark |
| Alex |
+------+
<strong>Explanation:</strong> 
According to orders 3 and 4 in the Orders table, it is easy to tell that only salesperson John and Pam have sales to company RED, so we report all the other names in the table salesperson.
</pre>
</div>

SQL

```mysql
select name
from SalesPerson
where sales_id not in (
    select distinct o.sales_id
    from orders o
    left join company c
    on o.com_id = c.com_id
    where o.com_id = (
        select com_id
        from company
        where name = "RED"
    )
);
```

## [1440. Evaluate Boolean Expression](https://leetcode.cn/problems/evaluate-boolean-expression/description/)

Description:

<div class="elfjS" data-track-load="description_content"><p>Table <code>Variables</code>:</p>

<pre>+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| name          | varchar |
| value         | int     |
+---------------+---------+
In SQL, name is the primary key for this table.
This table contains the stored variables and their values.
</pre>

<p>&nbsp;</p>

<p>Table <code>Expressions</code>:</p>

<pre>+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| left_operand  | varchar |
| operator      | enum    |
| right_operand | varchar |
+---------------+---------+
In SQL, (left_operand, operator, right_operand) is the primary key for this table.
This table contains a boolean expression that should be evaluated.
operator is an enum that takes one of the values ('&lt;', '&gt;', '=')
The values of left_operand and right_operand are guaranteed to be in the Variables table.
</pre>

<p>&nbsp;</p>

<p>Evaluate the boolean expressions in <code>Expressions</code> table.</p>

<p>Return the result table in <strong>any order</strong>.</p>

<p>The result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Variables table:
+------+-------+
| name | value |
+------+-------+
| x    | 66    |
| y    | 77    |
+------+-------+
Expressions table:
+--------------+----------+---------------+
| left_operand | operator | right_operand |
+--------------+----------+---------------+
| x            | &gt;        | y             |
| x            | &lt;        | y             |
| x            | =        | y             |
| y            | &gt;        | x             |
| y            | &lt;        | x             |
| x            | =        | x             |
+--------------+----------+---------------+
<strong>Output:</strong> 
+--------------+----------+---------------+-------+
| left_operand | operator | right_operand | value |
+--------------+----------+---------------+-------+
| x            | &gt;        | y             | false |
| x            | &lt;        | y             | true  |
| x            | =        | y             | false |
| y            | &gt;        | x             | true  |
| y            | &lt;        | x             | false |
| x            | =        | x             | true  |
+--------------+----------+---------------+-------+
<strong>Explanation:</strong> 
As shown, you need to find the value of each boolean expression in the table using the variables table.
</pre>
</div>

SQL

```mysql
select e.left_operand, e.operator, e.right_operand,
case when
    (case
        when operator = ">" then v.value > v2.value 
        when operator = "<" then v.value < v2.value 
        when operator = "=" then v.value = v2.value 
        end) = 0 then 'false' else 'true' end as value 
from Expressions e
left join Variables v
    on v.name = e.left_operand
left join Variables v2
    on v2.name = e.right_operand;

```

## [1212. Team Scores in Football Tournament](https://leetcode.cn/problems/team-scores-in-football-tournament/description/)

Description:
<div class="elfjS" data-track-load="description_content"><p>Table: <code>Teams</code></p>

<pre>+---------------+----------+
| Column Name   | Type     |
+---------------+----------+
| team_id       | int      |
| team_name     | varchar  |
+---------------+----------+
team_id is the column with unique values of this table.
Each row of this table represents a single football team.
</pre>

<p>&nbsp;</p>

<p>Table: <code>Matches</code></p>

<pre>+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| match_id      | int     |
| host_team     | int     |
| guest_team    | int     | 
| host_goals    | int     |
| guest_goals   | int     |
+---------------+---------+
match_id is the column of unique values of this table.
Each row is a record of a finished match between two different teams. 
Teams host_team and guest_team are represented by their IDs in the Teams table (team_id), and they scored host_goals and guest_goals goals, respectively.
</pre>

<p>&nbsp;</p>
You would like to compute the scores of all teams after all matches. Points are awarded as follows:

<ul>
	<li>A team receives <strong>three points</strong> if they win a match (i.e., Scored more goals than the opponent team).</li>
	<li>A team receives <strong>one point</strong> if they draw a match (i.e., Scored the same number of goals as the opponent team).</li>
	<li>A team receives <strong>no points</strong> if they lose a match (i.e., Scored fewer goals than the opponent team).</li>
</ul>

<p>Write a solution that selects the <code>team_id</code>, <code>team_name</code> and <code>num_points</code> of each team in the tournament after all described matches.</p>

<p>Return the result table ordered by <code>num_points</code> <strong>in decreasing order</strong>. In case of a tie, order the records by <code>team_id</code> <strong>in increasing order</strong>.</p>

<p>The result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Teams table:
+-----------+--------------+
| team_id   | team_name    |
+-----------+--------------+
| 10        | Leetcode FC  |
| 20        | NewYork FC   |
| 30        | Atlanta FC   |
| 40        | Chicago FC   |
| 50        | Toronto FC   |
+-----------+--------------+
Matches table:
+------------+--------------+---------------+-------------+--------------+
| match_id   | host_team    | guest_team    | host_goals  | guest_goals  |
+------------+--------------+---------------+-------------+--------------+
| 1          | 10           | 20            | 3           | 0            |
| 2          | 30           | 10            | 2           | 2            |
| 3          | 10           | 50            | 5           | 1            |
| 4          | 20           | 30            | 1           | 0            |
| 5          | 50           | 30            | 1           | 0            |
+------------+--------------+---------------+-------------+--------------+
<strong>Output:</strong> 
+------------+--------------+---------------+
| team_id    | team_name    | num_points    |
+------------+--------------+---------------+
| 10         | Leetcode FC  | 7             |
| 20         | NewYork FC   | 3             |
| 50         | Toronto FC   | 3             |
| 30         | Atlanta FC   | 1             |
| 40         | Chicago FC   | 0             |
+------------+--------------+---------------+
</pre>
</div>

SQL

```mysql
select t.team_id, t.team_name, ifnull(sum(m.num_points), 0) as num_points
from 
(
    select team_id, sum(num_points) as num_points
    from (
        select host_team as team_id,
            case
                when host_goals > guest_goals then 3
                when host_goals = guest_goals then 1
                when host_goals < guest_goals then 0
            end as num_points
        from matches
        union all
        select guest_team as team_id,
        case 
            when host_goals < guest_goals then 3
            when host_goals = guest_goals then 1
            when host_goals > guest_goals then 0
        end as num_points
        from matches
) m
right join teams t
on t.team_id = m.team_id
group by t.team_id
order by num_points desc, t.team_id asc
```

## [1571. Warehouse Manager](https://leetcode.cn/problems/warehouse-manager/description/)

Description:

<div class="elfjS" data-track-load="description_content"><p>Table: <code>Warehouse</code></p>

<pre>+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| name         | varchar |
| product_id   | int     |
| units        | int     |
+--------------+---------+
(name, product_id) is the primary key (combination of columns with unique values) for this table.
Each row of this table contains the information of the products in each warehouse.
</pre>

<p>&nbsp;</p>

<p>Table: <code>Products</code></p>

<pre>+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| product_name  | varchar |
| Width         | int     |
| Length        | int     |
| Height        | int     |
+---------------+---------+
product_id is the primary key (column with unique values) for this table.
Each row of this table contains information about the product dimensions (Width, Lenght, and Height) in feets of each product.
</pre>

<p>&nbsp;</p>

<p>Write a solution to report the number of cubic feet of <strong>volume </strong>the inventory occupies in each warehouse.</p>

<p>Return the result table in <strong>any order</strong>.</p>

<p>The query result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Warehouse table:
+------------+--------------+-------------+
| name       | product_id   | units       |
+------------+--------------+-------------+
| LCHouse1   | 1            | 1           |
| LCHouse1   | 2            | 10          |
| LCHouse1   | 3            | 5           |
| LCHouse2   | 1            | 2           |
| LCHouse2   | 2            | 2           |
| LCHouse3   | 4            | 1           |
+------------+--------------+-------------+
Products table:
+------------+--------------+------------+----------+-----------+
| product_id | product_name | Width      | Length   | Height    |
+------------+--------------+------------+----------+-----------+
| 1          | LC-TV        | 5          | 50       | 40        |
| 2          | LC-KeyChain  | 5          | 5        | 5         |
| 3          | LC-Phone     | 2          | 10       | 10        |
| 4          | LC-T-Shirt   | 4          | 10       | 20        |
+------------+--------------+------------+----------+-----------+
<strong>Output:</strong> 
+----------------+------------+
| warehouse_name | volume     | 
+----------------+------------+
| LCHouse1       | 12250      | 
| LCHouse2       | 20250      |
| LCHouse3       | 800        |
+----------------+------------+
<strong>Explanation:</strong> 
Volume of product_id = 1 (LC-TV), 5x50x40 = 10000
Volume of product_id = 2 (LC-KeyChain), 5x5x5 = 125 
Volume of product_id = 3 (LC-Phone), 2x10x10 = 200
Volume of product_id = 4 (LC-T-Shirt), 4x10x20 = 800
LCHouse1: 1 unit of LC-TV + 10 units of LC-KeyChain + 5 units of LC-Phone.
          Total volume: 1*10000 + 10*125  + 5*200 = 12250 cubic feet
LCHouse2: 2 units of LC-TV + 2 units of LC-KeyChain.
          Total volume: 2*10000 + 2*125 = 20250 cubic feet
LCHouse3: 1 unit of LC-T-Shirt.
          Total volume: 1*800 = 800 cubic feet.
</pre>
</div>

SQL

```mysql
select name as warehouse_name, sum(units * volume) as volume
from warehouse w
join
(
    select product_id, (width * length * height) as volume
    from products
) v
on w.product_id = v.product_id
group by name

```

## [1741. Find Total Time Spent by Each Employee](https://leetcode.cn/problems/find-total-time-spent-by-each-employee/description/)

Description:

<div class="elfjS" data-track-load="description_content"><p>Table: <code>Employees</code></p>

<pre>+-------------+------+
| Column Name | Type |
+-------------+------+
| emp_id      | int  |
| event_day   | date |
| in_time     | int  |
| out_time    | int  |
+-------------+------+
(emp_id, event_day, in_time) is the primary key (combinations of columns with unique values) of this table.
The table shows the employees' entries and exits in an office.
event_day is the day at which this event happened, in_time is the minute at which the employee entered the office, and out_time is the minute at which they left the office.
in_time and out_time are between 1 and 1440.
It is guaranteed that no two events on the same day intersect in time, and in_time &lt; out_time.
</pre>

<p>&nbsp;</p>

<p>Write a solution to calculate the total time <strong>in minutes</strong> spent by each employee on each day at the office. Note that within one day, an employee can enter and leave more than once. The time spent in the office for a single entry is <code>out_time - in_time</code>.</p>

<p>Return the result table in <strong>any order</strong>.</p>

<p>The result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Employees table:
+--------+------------+---------+----------+
| emp_id | event_day  | in_time | out_time |
+--------+------------+---------+----------+
| 1      | 2020-11-28 | 4       | 32       |
| 1      | 2020-11-28 | 55      | 200      |
| 1      | 2020-12-03 | 1       | 42       |
| 2      | 2020-11-28 | 3       | 33       |
| 2      | 2020-12-09 | 47      | 74       |
+--------+------------+---------+----------+
<strong>Output:</strong> 
+------------+--------+------------+
| day        | emp_id | total_time |
+------------+--------+------------+
| 2020-11-28 | 1      | 173        |
| 2020-11-28 | 2      | 30         |
| 2020-12-03 | 1      | 41         |
| 2020-12-09 | 2      | 27         |
+------------+--------+------------+
<strong>Explanation:</strong> 
Employee 1 has three events: two on day 2020-11-28 with a total of (32 - 4) + (200 - 55) = 173, and one on day 2020-12-03 with a total of (42 - 1) = 41.
Employee 2 has two events: one on day 2020-11-28 with a total of (33 - 3) = 30, and one on day 2020-12-09 with a total of (74 - 47) = 27.
</pre>
</div>

SQL

```mysql
select event_day as day, emp_id, sum(out_time) - sum(in_time) as total_time
from employees
group by
    event_day, emp_id
```

## [1173. Immediate Food Delivery I](https://leetcode.cn/problems/immediate-food-delivery-i/description/)

Description:

<div class="elfjS" data-track-load="description_content"><p>Table: <code>Delivery</code></p>

<pre>+-----------------------------+---------+
| Column Name                 | Type    |
+-----------------------------+---------+
| delivery_id                 | int     |
| customer_id                 | int     |
| order_date                  | date    |
| customer_pref_delivery_date | date    |
+-----------------------------+---------+
delivery_id is the primary key (column with unique values) of this table.
The table holds information about food delivery to customers that make orders at some date and specify a preferred delivery date (on the same order date or after it).
</pre>

<p>&nbsp;</p>

<p>If the customer's preferred delivery date is the same as the order date, then the order is called <strong>immediate;</strong> otherwise, it is called <strong>scheduled</strong>.</p>

<p>Write a solution to find the percentage of immediate orders in the table, <strong>rounded to 2 decimal places</strong>.</p>

<p>The result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Delivery table:
+-------------+-------------+------------+-----------------------------+
| delivery_id | customer_id | order_date | customer_pref_delivery_date |
+-------------+-------------+------------+-----------------------------+
| 1           | 1           | 2019-08-01 | 2019-08-02                  |
| 2           | 5           | 2019-08-02 | 2019-08-02                  |
| 3           | 1           | 2019-08-11 | 2019-08-11                  |
| 4           | 3           | 2019-08-24 | 2019-08-26                  |
| 5           | 4           | 2019-08-21 | 2019-08-22                  |
| 6           | 2           | 2019-08-11 | 2019-08-13                  |
+-------------+-------------+------------+-----------------------------+
<strong>Output:</strong> 
+----------------------+
| immediate_percentage |
+----------------------+
| 33.33                |
+----------------------+
<strong>Explanation:</strong> The orders with delivery id 2 and 3 are immediate while the others are scheduled.
</pre>
</div>

SQL

```mysql
select round(immediate_num/total * 100, 2) as immediate_percentage 
from (
    select avg()
    from (
        select *, 
            case
                when customer_pref_delivery_date = order_date then 1
                else 0
                end as immediate
        from delivery
    ) i
) t

official:
SELECT ROUND(
    100 * AVG(order_date = customer_pref_delivery_date), 
    2) AS immediate_percentage
FROM 
    Delivery;

```

## []()

Description:

<div class="elfjS" data-track-load="description_content"><p>Table: <code>Sales</code></p>

<pre>+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| sale_date     | date    |
| fruit         | enum    | 
| sold_num      | int     | 
+---------------+---------+
(sale_date, fruit) is the primary key (combination of columns with unique values) of this table.
This table contains the sales of "apples" and "oranges" sold each day.
</pre>

<p>&nbsp;</p>

<p>Write a solution to report the difference between the number of <strong>apples</strong> and <strong>oranges</strong> sold each day.</p>

<p>Return the result table <strong>ordered</strong> by <code>sale_date</code>.</p>

<p>The result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Sales table:
+------------+------------+-------------+
| sale_date  | fruit      | sold_num    |
+------------+------------+-------------+
| 2020-05-01 | apples     | 10          |
| 2020-05-01 | oranges    | 8           |
| 2020-05-02 | apples     | 15          |
| 2020-05-02 | oranges    | 15          |
| 2020-05-03 | apples     | 20          |
| 2020-05-03 | oranges    | 0           |
| 2020-05-04 | apples     | 15          |
| 2020-05-04 | oranges    | 16          |
+------------+------------+-------------+
<strong>Output:</strong> 
+------------+--------------+
| sale_date  | diff         |
+------------+--------------+
| 2020-05-01 | 2            |
| 2020-05-02 | 0            |
| 2020-05-03 | 20           |
| 2020-05-04 | -1           |
+------------+--------------+
<strong>Explanation:</strong> 
Day 2020-05-01, 10 apples and 8 oranges were sold (Difference  10 - 8 = 2).
Day 2020-05-02, 15 apples and 15 oranges were sold (Difference 15 - 15 = 0).
Day 2020-05-03, 20 apples and 0 oranges were sold (Difference 20 - 0 = 20).
Day 2020-05-04, 15 apples and 16 oranges were sold (Difference 15 - 16 = -1).
</pre>
</div>

SQL

```mysql
select a.sale_date, (apples - oranges) as diff
from 
(
    (select sale_date, sold_num as apples
        from sales
        where fruit = "apples") a
    join
    (select sale_date, sold_num as oranges
        from sales
        where fruit = "oranges") b
    on a.sale_date = b.sale_date
)
group by
    a.sale_date

# Write your MySQL query statement below
select sale_date, sum(if(fruit = 'apples', sold_num, -sold_num)) as diff
from Sales
group by sale_date
order by sale_date;

```

## [1699. Number of Calls Between Two Persons](https://leetcode.cn/problems/number-of-calls-between-two-persons/description/)

Description:

<div class="elfjS" data-track-load="description_content"><p>Table: <code>Calls</code></p>

<pre>+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| from_id     | int     |
| to_id       | int     |
| duration    | int     |
+-------------+---------+
This table does not have a primary key (column with unique values), it may contain duplicates.
This table contains the duration of a phone call between from_id and to_id.
from_id != to_id
</pre>

<p>&nbsp;</p>

<p>Write a solution&nbsp;to report the number of calls and the total call duration between each pair of distinct persons <code>(person1, person2)</code> where <code>person1 &lt; person2</code>.</p>

<p>Return the result table in <strong>any order</strong>.</p>

<p>The result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Calls table:
+---------+-------+----------+
| from_id | to_id | duration |
+---------+-------+----------+
| 1       | 2     | 59       |
| 2       | 1     | 11       |
| 1       | 3     | 20       |
| 3       | 4     | 100      |
| 3       | 4     | 200      |
| 3       | 4     | 200      |
| 4       | 3     | 499      |
+---------+-------+----------+
<strong>Output:</strong> 
+---------+---------+------------+----------------+
| person1 | person2 | call_count | total_duration |
+---------+---------+------------+----------------+
| 1       | 2       | 2          | 70             |
| 1       | 3       | 1          | 20             |
| 3       | 4       | 4          | 999            |
+---------+---------+------------+----------------+
<strong>Explanation:</strong> 
Users 1 and 2 had 2 calls and the total duration is 70 (59 + 11).
Users 1 and 3 had 1 call and the total duration is 20.
Users 3 and 4 had 4 calls and the total duration is 999 (100 + 200 + 200 + 499).
</pre>
</div>

SQL

```mysql
select 
    case 
        when from_id > to_id then to_id
        else from_id
    end as person1,
    case 
        when from_id > to_id then from_id
        else to_id
    end as person2,
    count(duration) as call_count,
    sum(duration) as total_duration
from calls
group by
    person1, person2
```

## [1511. Customer Order Frequency](https://leetcode.cn/problems/customer-order-frequency/description/)

Description:

<div class="elfjS" data-track-load="description_content"><p>Table: <code>Customers</code></p>

<pre>+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| customer_id   | int     |
| name          | varchar |
| country       | varchar |
+---------------+---------+
customer_id is the column with unique values for this table.
This table contains information about the customers in the company.
</pre>

<p>&nbsp;</p>

<p>Table: <code>Product</code></p>

<pre>+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| description   | varchar |
| price         | int     |
+---------------+---------+
product_id is the column with unique values for this table.
This table contains information on the products in the company.
price is the product cost.
</pre>

<p>&nbsp;</p>

<p>Table: <code>Orders</code></p>

<pre>+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| order_id      | int     |
| customer_id   | int     |
| product_id    | int     |
| order_date    | date    |
| quantity      | int     |
+---------------+---------+
order_id is the column with unique values for this table.
This table contains information on customer orders.
customer_id is the id of the customer who bought "quantity" products with id "product_id".
Order_date is the date in format ('YYYY-MM-DD') when the order was shipped.
</pre>

<p>&nbsp;</p>

<p>Write a solution to report the <code>customer_id</code> and <code>customer_name</code> of customers who have spent at least <code>$100</code> in each month of <strong>June and July 2020</strong>.</p>

<p>Return the result table in <strong>any order</strong>.</p>

<p>The&nbsp;result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Customers table:
+--------------+-----------+-------------+
| customer_id  | name      | country     |
+--------------+-----------+-------------+
| 1            | Winston   | USA         |
| 2            | Jonathan  | Peru        |
| 3            | Moustafa  | Egypt       |
+--------------+-----------+-------------+
Product table:
+--------------+-------------+-------------+
| product_id   | description | price       |
+--------------+-------------+-------------+
| 10           | LC Phone    | 300         |
| 20           | LC T-Shirt  | 10          |
| 30           | LC Book     | 45          |
| 40           | LC Keychain | 2           |
+--------------+-------------+-------------+
Orders table:
+--------------+-------------+-------------+-------------+-----------+
| order_id     | customer_id | product_id  | order_date  | quantity  |
+--------------+-------------+-------------+-------------+-----------+
| 1            | 1           | 10          | 2020-06-10  | 1         |
| 2            | 1           | 20          | 2020-07-01  | 1         |
| 3            | 1           | 30          | 2020-07-08  | 2         |
| 4            | 2           | 10          | 2020-06-15  | 2         |
| 5            | 2           | 40          | 2020-07-01  | 10        |
| 6            | 3           | 20          | 2020-06-24  | 2         |
| 7            | 3           | 30          | 2020-06-25  | 2         |
| 9            | 3           | 30          | 2020-05-08  | 3         |
+--------------+-------------+-------------+-------------+-----------+
<strong>Output:</strong> 
+--------------+------------+
| customer_id  | name       |  
+--------------+------------+
| 1            | Winston    |
+--------------+------------+
<strong>Explanation:</strong> 
Winston spent $300 (300 * 1) in June and $100 ( 10 * 1 + 45 * 2) in July 2020.
Jonathan spent $600 (300 * 2) in June and $20 ( 2 * 10) in July 2020.
Moustafa spent $110 (10 * 2 + 45 * 2) in June and $0 in July 2020.
</pre>
</div>

SQL

```mysql
select c.customer_id, c.name
from 
(
    select customer_id, sum(quantity * price) as total_spent, date_format(order_date, "%Y-%m") as month
    from orders o
    join product p
    on o.product_id = p.product_id
    where 
        date_format(order_date, "%Y-%m") = "2020-06" or date_format(order_date, "%Y-%m") = "2020-07"
    group by customer_id, month
    having 
        total_spent >= 100
) a
join customers c
on c.customer_id = a.customer_id
group by c.customer_id
having
    count(c.customer_id) > 1
```

## [1495. Friendly Movies Streamed Last Month](https://leetcode.cn/problems/friendly-movies-streamed-last-month/description/)

Description:

<div class="elfjS" data-track-load="description_content"><p>Table: <code>TVProgram</code></p>

<pre>+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| program_date  | date    |
| content_id    | int     |
| channel       | varchar |
+---------------+---------+
(program_date, content_id) is the primary key (combination of columns with unique values) for this table.
This table contains information of the programs on the TV.
content_id is the id of the program in some channel on the TV.</pre>

<p>&nbsp;</p>

<p>Table: <code>Content</code></p>

<pre>+------------------+---------+
| Column Name      | Type    |
+------------------+---------+
| content_id       | varchar |
| title            | varchar |
| Kids_content     | enum    |
| content_type     | varchar |
+------------------+---------+
content_id is the primary key (column with unique values) for this table.
Kids_content is an ENUM (category) of types ('Y', 'N') where: 
'Y' means is content for kids otherwise 'N' is not content for kids.
content_type is the category of the content as movies, series, etc.
</pre>

<p>&nbsp;</p>

<p>Write a solution to report the distinct titles of the kid-friendly movies streamed in <strong>June 2020</strong>.</p>

<p>Return the result table in <strong>any order</strong>.</p>

<p>The result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
TVProgram table:
+--------------------+--------------+-------------+
| program_date       | content_id   | channel     |
+--------------------+--------------+-------------+
| 2020-06-10 08:00   | 1            | LC-Channel  |
| 2020-05-11 12:00   | 2            | LC-Channel  |
| 2020-05-12 12:00   | 3            | LC-Channel  |
| 2020-05-13 14:00   | 4            | Disney Ch   |
| 2020-06-18 14:00   | 4            | Disney Ch   |
| 2020-07-15 16:00   | 5            | Disney Ch   |
+--------------------+--------------+-------------+
Content table:
+------------+----------------+---------------+---------------+
| content_id | title          | Kids_content  | content_type  |
+------------+----------------+---------------+---------------+
| 1          | Leetcode Movie | N             | Movies        |
| 2          | Alg. for Kids  | Y             | Series        |
| 3          | Database Sols  | N             | Series        |
| 4          | Aladdin        | Y             | Movies        |
| 5          | Cinderella     | Y             | Movies        |
+------------+----------------+---------------+---------------+
<strong>Output:</strong> 
+--------------+
| title        |
+--------------+
| Aladdin      |
+--------------+
<strong>Explanation:</strong> 
"Leetcode Movie" is not a content for kids.
"Alg. for Kids" is not a movie.
"Database Sols" is not a movie
"Alladin" is a movie, content for kids and was streamed in June 2020.
"Cinderella" was not streamed in June 2020.
</pre>
</div>

SQL

```mysql
select distinct c.title
from tvprogram t
join content c
on t.content_id = c.content_id 
where
    date_format(t.program_date, "%Y-%m") = "2020-06" and
    kids_content = "Y" and
    content_type = "Movies"

```

## [1501. Countries You Can Safely Invest In](https://leetcode.cn/problems/countries-you-can-safely-invest-in/description/)

Description:

<div class="elfjS" data-track-load="description_content"><p>Table <code>Person</code>:</p>

<pre>+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| id             | int     |
| name           | varchar |
| phone_number   | varchar |
+----------------+---------+
id is the column of unique values for this table.
Each row of this table contains the name of a person and their phone number.
Phone number will be in the form 'xxx-yyyyyyy' where xxx is the country code (3 characters) and yyyyyyy is the phone number (7 characters) where x and y are digits. Both can contain leading zeros.
</pre>

<p>&nbsp;</p>

<p>Table <code>Country</code>:</p>

<pre>+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| name           | varchar |
| country_code   | varchar |
+----------------+---------+
country_code is the column of unique values for this table.
Each row of this table contains the country name and its code. country_code will be in the form 'xxx' where x is digits.
</pre>

<p>&nbsp;</p>

<p>Table <code>Calls</code>:</p>

<pre>+-------------+------+
| Column Name | Type |
+-------------+------+
| caller_id   | int  |
| callee_id   | int  |
| duration    | int  |
+-------------+------+
This table may contain duplicate rows.
Each row of this table contains the caller id, callee id and the duration of the call in minutes. caller_id != callee_id
</pre>

<p>&nbsp;</p>

<p>A telecommunications company wants to invest in new countries. The company intends to invest in the countries where the average call duration of the calls in this country is strictly greater than the global average call duration.</p>

<p>Write a solution to find the countries where this company can invest.</p>

<p>Return the result table in <strong>any order</strong>.</p>

<p>The result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Person table:
+----+----------+--------------+
| id | name     | phone_number |
+----+----------+--------------+
| 3  | Jonathan | 051-1234567  |
| 12 | Elvis    | 051-7654321  |
| 1  | Moncef   | 212-1234567  |
| 2  | Maroua   | 212-6523651  |
| 7  | Meir     | 972-1234567  |
| 9  | Rachel   | 972-0011100  |
+----+----------+--------------+
Country table:
+----------+--------------+
| name     | country_code |
+----------+--------------+
| Peru     | 051          |
| Israel   | 972          |
| Morocco  | 212          |
| Germany  | 049          |
| Ethiopia | 251          |
+----------+--------------+
Calls table:
+-----------+-----------+----------+
| caller_id | callee_id | duration |
+-----------+-----------+----------+
| 1         | 9         | 33       |
| 2         | 9         | 4        |
| 1         | 2         | 59       |
| 3         | 12        | 102      |
| 3         | 12        | 330      |
| 12        | 3         | 5        |
| 7         | 9         | 13       |
| 7         | 1         | 3        |
| 9         | 7         | 1        |
| 1         | 7         | 7        |
+-----------+-----------+----------+
<strong>Output:</strong> 
+----------+
| country  |
+----------+
| Peru     |
+----------+
<strong>Explanation:</strong> 
The average call duration for Peru is (102 + 102 + 330 + 330 + 5 + 5) / 6 = 145.666667
The average call duration for Israel is (33 + 4 + 13 + 13 + 3 + 1 + 1 + 7) / 8 = 9.37500
The average call duration for Morocco is (33 + 4 + 59 + 59 + 3 + 7) / 6 = 27.5000 
Global call duration average = (2 * (33 + 4 + 59 + 102 + 330 + 5 + 13 + 3 + 1 + 7)) / 20 = 55.70000
Since Peru is the only country where the average call duration is greater than the global average, it is the only recommended country.
</pre>
</div>

SQL

```mysql
with person_country as (
    select p.id, c.name as country
    from person p
    join country c
        on c.country_code = left(p.phone_number, 3)
)

select country
from
(
    select ca.duration, pc.country
    from calls ca
    join person_country pc
        on ca.caller_id = pc.id or ca.callee_id = pc.id
    -- union all
    -- select ca.duration, pc.country
    -- from calls ca
    -- join person_country pc
    --     on ca.callee_id = pc.id
) total
group by country
having
    avg(duration) > (
        select avg(duration)
        from calls
    )
```

## [603. Consecutive Available Seats](https://leetcode.cn/problems/consecutive-available-seats/description/)

Description:

<div class="elfjS" data-track-load="description_content"><p>Table: <code>Cinema</code></p>

<pre>+-------------+------+
| Column Name | Type |
+-------------+------+
| seat_id     | int  |
| free        | bool |
+-------------+------+
seat_id is an auto-increment column for this table.
Each row of this table indicates whether the i<sup>th</sup> seat is free or not. 1 means free while 0 means occupied.
</pre>

<p>&nbsp;</p>

<p>Find all the consecutive available seats in the cinema.</p>

<p>Return the result table <strong>ordered</strong> by <code>seat_id</code> <strong>in ascending order</strong>.</p>

<p>The test cases are generated so that more than two seats are consecutively available.</p>

<p>The result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> 
Cinema table:
+---------+------+
| seat_id | free |
+---------+------+
| 1       | 1    |
| 2       | 0    |
| 3       | 1    |
| 4       | 1    |
| 5       | 1    |
+---------+------+
<strong>Output:</strong> 
+---------+
| seat_id |
+---------+
| 3       |
| 4       |
| 5       |
+---------+
</pre>
</div>

SQL

```mysql
应该用笛卡尔积
select distinct a.seat_id
from cinema a
join cinema b
on a.seat_id = b.seat_id - 1 or a.seat_id = b.seat_id + 1
where a.free = 1 and ifnull(b.free, a.free) = 1
order by a.seat_id
```

## []()

Description:

SQL

```mysql

```


