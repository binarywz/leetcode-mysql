# LeetCode MySQL

### [175. 组合两个表](https://leetcode-cn.com/problems/combine-two-tables/)

```mysql
-- SQL架构
Create table Person (PersonId int, FirstName varchar(255), LastName varchar(255));
Create table Address (AddressId int, PersonId int, City varchar(255), State varchar(255));
Truncate table Person;
insert into Person (PersonId, LastName, FirstName) values ('1', 'Wang', 'Allen');
Truncate table Address;
insert into Address (AddressId, PersonId, City, State) values ('1', '2', 'New York City', 'New York');

-- 表1 Person
+-------------+---------+
| 列名         | 类型     |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
PersonId 是上表主键

-- 表2 Address
+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+--------+
AddressId 是上表主键 

-- 编写一个 SQL 查询，满足条件：无论 person 是否有地址信息，都需要基于上述两表提供 person 的以下信息：
FirstName, LastName, City, State

-- SQL语句
SELECT FirstName, LastName, City, State
FROM Person p LEFT JOIN Address a ON p.PersonId = a.PersonId;

SELECT A.FirstName, A.LastName, B.City, B.State
FROM Person A LEFT JOIN (SELECT DISTINCT PersonId, City, State FROM Address) B
ON A.PersonId=B.PersonId;
```

### [176. 第二高的薪水](https://leetcode-cn.com/problems/second-highest-salary/)

```mysql
-- SQL架构
Create table If Not Exists Employee (Id int, Salary int);
Truncate table Employee;
insert into Employee (Id, Salary) values ('1', '100');
insert into Employee (Id, Salary) values ('2', '200');
insert into Employee (Id, Salary) values ('3', '300');

-- 编写一个 SQL 查询，获取 Employee 表中第二高的薪水（Salary）
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+

-- 例如上述 Employee 表，SQL查询应该返回 200 作为第二高的薪水。如果不存在第二高的薪水，那么查询应返回 null
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+

-- SQL语句
SELECT
(CASE WHEN MAX(Salary) IS NOT NULL THEN MAX(Salary) ELSE NULL END) SecondHighestSalary
FROM employee WHERE Salary != (SELECT MAX(Salary) FROM employee);

SELECT 
(SELECT DISTINCT Salary
    FROM Employee
    ORDER BY Salary DESC
    LIMIT 1 OFFSET 1) AS SecondHighestSalary;
```

### [177. 第N高的薪水](https://leetcode-cn.com/problems/nth-highest-salary/)

```mysql
-- SQL架构
Create table If Not Exists Employee (Id int, Salary int);
Truncate table Employee;
insert into Employee (Id, Salary) values ('1', '100');
insert into Employee (Id, Salary) values ('2', '200');
insert into Employee (Id, Salary) values ('3', '300');

-- 编写一个SQL查询，获取Employee表中第n高的薪水（Salary）
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+

-- 例如上述Employee表，n = 2时，应返回第二高的薪水200。如果不存在第n高的薪水，那么查询应返回 null
+------------------------+
| getNthHighestSalary(2) |
+------------------------+
| 200                    |
+------------------------+

-- MySQL函数
CREATE FUNCTION 函数名([参数列表]) RETURNS 数据类型
BEGIN
 sql语句;
 RETURN 值;
END;

-- SQL语句
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  SET N = N - 1;
  RETURN (
      # Write your MySQL query statement below.
      SELECT DISTINCT Salary FROM Employee ORDER BY Salary DESC LIMIT N, 1
  );
END

-- 不知道为什么加一个SELECT查询速度会变快
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  SET N = N - 1;
  RETURN (
      # Write your MySQL query statement below.
      SELECT (SELECT DISTINCT Salary FROM Employee ORDER BY Salary DESC LIMIT N, 1)
  );
END
```

### [178. 分数排名](https://leetcode-cn.com/problems/rank-scores/)

```mysql
-- SQL架构
Create table If Not Exists Scores (Id int, Score DECIMAL(3,2));
Truncate table Scores;
insert into Scores (Id, Score) values ('1', '3.5');
insert into Scores (Id, Score) values ('2', '3.65');
insert into Scores (Id, Score) values ('3', '4.0');
insert into Scores (Id, Score) values ('4', '3.85');
insert into Scores (Id, Score) values ('5', '4.0');
insert into Scores (Id, Score) values ('6', '3.65');

-- 编写一个 SQL 查询来实现分数排名。如果两个分数相同，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有“间隔”。
+----+-------+
| Id | Score |
+----+-------+
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |
+----+-------+

-- 例如，根据上述给定的 Scores 表，你的查询应该返回（按分数从高到低排列）：
+-------+------+
| Score | Rank |
+-------+------+
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |
+-------+------+

-- 在CASE WHEN的返回值后面+0，即主动把CASE WHEN的返回值变为数字类型:
SELECT Score,
(CASE WHEN @pre = Score THEN @cur 
WHEN (@pre := Score) IS NOT NULL THEN @cur := @cur + 1 END + 0) AS Rank
FROM Scores s, (SELECT @cur := 0, @pre := NULL) r ORDER BY Score DESC;
```

### [180. 连续出现的数字](https://leetcode-cn.com/problems/consecutive-numbers/)

```mysql
-- SQL架构
Create table If Not Exists Logs (Id int, Num int);
Truncate table Logs;
insert into Logs (Id, Num) values ('1', '1');
insert into Logs (Id, Num) values ('2', '1');
insert into Logs (Id, Num) values ('3', '1');
insert into Logs (Id, Num) values ('4', '2');
insert into Logs (Id, Num) values ('5', '1');
insert into Logs (Id, Num) values ('6', '2');
insert into Logs (Id, Num) values ('7', '2');

-- 编写一个SQL查询，查找所有至少连续出现三次的数字
+----+-----+
| Id | Num |
+----+-----+
| 1  |  1  |
| 2  |  1  |
| 3  |  1  |
| 4  |  2  |
| 5  |  1  |
| 6  |  2  |
| 7  |  2  |
+----+-----+

-- 例如，给定上面的 Logs 表， 1 是唯一连续出现至少三次的数字
+-----------------+
| ConsecutiveNums |
+-----------------+
| 1               |
+-----------------+

-- SQL语句
SELECT DISTINCT l3.Num ConsecutiveNums
FROM logs l1 JOIN logs l2 ON l2.Id = l1.Id + 1 AND l2.Num = l1.Num
JOIN logs l3 ON l3.Id = l2.Id + 1 AND l2.Num = l3.Num;

-- MySQL用户变量
-- 可以先在用户变量中保存值然后在以后引用它，这样可以将值从一个语句传递到另一个语句。用户变量与连接相关，也就是说，一个客户端定义的变量不能被其他客户端看到或使用，当客户端退出时，该客户端连接的所有变量将自动释放。用户变量的形式为@var_name，变量名对大小写不敏感。
-- 设置用户变量的一个途径是执行SET语句：SET @var_name = expr，对于SET，可以使用=或:=作为分配符，分配给每个变量的expr可以为整数、实数、字符串或者NULL值。
-- 也可以使用语句代替SET来为用户变量分配一个值，在这种情况下，分配符必须为:=，因为在非SET语句中=被视为一个比较操作符。
SELECT DISTINCT Num AS ConsecutiveNums
FROM (
    SELECT Num, 
        CASE 
        WHEN @prev = Num THEN @count := @count + 1
        WHEN (@prev := Num) IS NOT NULL THEN @count := 1
        END AS CNT
        FROM Logs, (SELECT @prev := NULL,@count := NULL) as t
) AS temp
WHERE temp.CNT >= 3;
```

### [181. 超过经理收入的员工](https://leetcode-cn.com/problems/employees-earning-more-than-their-managers/)

```mysql
-- SQL架构
Create table If Not Exists Employee (Id int, Name varchar(255), Salary int, ManagerId int);
Truncate table Employee;
insert into Employee (Id, Name, Salary, ManagerId) values ('1', 'Joe', '70000', '3');
insert into Employee (Id, Name, Salary, ManagerId) values ('2', 'Henry', '80000', '4');
insert into Employee (Id, Name, Salary, ManagerId) values ('3', 'Sam', '60000', NULL);
insert into Employee (Id, Name, Salary, ManagerId) values ('4', 'Max', '90000', NULL);

-- Employee表包含所有员工，他们的经理也属于员工。每个员工都有一个Id，此外还有一列对应员工的经理的Id
+----+-------+--------+-----------+
| Id | Name  | Salary | ManagerId |
+----+-------+--------+-----------+
| 1  | Joe   | 70000  | 3         |
| 2  | Henry | 80000  | 4         |
| 3  | Sam   | 60000  | NULL      |
| 4  | Max   | 90000  | NULL      |
+----+-------+--------+-----------+

-- SQL语句
SELECT e1.Name AS Employee FROM employee e1 LEFT JOIN employee e2 ON e1.ManagerId = e2.Id WHERE e1.Salary > e2.Salary;

SELECT 
    e.Name Employee
FROM Employee e JOIN (SELECT DISTINCT Id, Salary FROM Employee) m  
    ON e.ManagerId  = m.Id 
WHERE e.Salary  > m.Salary;
```

### [182. 查找重复的电子邮箱](https://leetcode-cn.com/problems/duplicate-emails/)

```mysql
-- SQL架构
Create table If Not Exists Person (Id int, Email varchar(255));
Truncate table Person;
insert into Person (Id, Email) values ('1', 'a@b.com');
insert into Person (Id, Email) values ('2', 'c@d.com');
insert into Person (Id, Email) values ('3', 'a@b.com');

-- 编写一个 SQL 查询，查找 Person 表中所有重复的电子邮箱
+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+

-- 根据以上输入，，你的查询应返回以下结果：
-- 说明：所有电子邮箱都是小写字母。
+---------+
| Email   |
+---------+
| a@b.com |
+---------+

-- SQL语句
SELECT e.em AS Email FROM
(SELECT CASE WHEN COUNT(*) > 1 THEN Email END AS em
FROM Person GROUP BY Email) AS e
WHERE e.em IS NOT NULL;

SELECT Email FROM Person GROUP BY Email HAVING COUNT(1) > 1;
```

### [183. 从不订购的客户](https://leetcode-cn.com/problems/customers-who-never-order/)

```mysql
-- SQL架构
Create table If Not Exists Customers (Id int, Name varchar(255));
Create table If Not Exists Orders (Id int, CustomerId int);
Truncate table Customers;
insert into Customers (Id, Name) values ('1', 'Joe');
insert into Customers (Id, Name) values ('2', 'Henry');
insert into Customers (Id, Name) values ('3', 'Sam');
insert into Customers (Id, Name) values ('4', 'Max');
Truncate table Orders;
insert into Orders (Id, CustomerId) values ('1', '3');
insert into Orders (Id, CustomerId) values ('2', '1');

-- 某网站包含两个表，Customers表和Orders表。编写一个SQL查询，找出所有从不订购任何东西的客户
-- Customers表：
+----+-------+
| Id | Name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
-- Orders表：
+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+

-- 例如给定上述表格，你的查询应返回：
+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+

-- SQL语句
SELECT Name AS Customers FROM customers WHERE Id NOT IN (SELECT CustomerId FROM orders);

-- 使用完全限定列名查询速度会快
SELECT c.Name AS Customers FROM Customers c WHERE c.Id NOT IN (SELECT CustomerId FROM orders);

SELECT c.Name Customer FROM Customers c LEFT JOIN Orders o ON c.Id = o.CustomerId WHERE o.Id IS NULL;
```

