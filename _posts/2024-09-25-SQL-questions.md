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

### Description
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

### SQL

```mysql
select customer_id from customers where revenue > 0 and year = 2021
```

## [🟢 183. Customers Who Never Order](https://leetcode.cn/problems/customers-who-never-order/)

### Des:
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

### SQL

```mysql
select name as Customers
from customers
where id not in (
    select customerId from orders
)
```

## [🟢 1873. Calculate Special Bonus](https://leetcode.cn/problems/calculate-special-bonus/description/)

### Description:
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

### SQL

```mysql
select employee_id,
    case
        when name not like "m%" and employee_id % 2 != 0 then salary
        else 0 
        end as bonus
from Employees 
order by employee_id asc
```
