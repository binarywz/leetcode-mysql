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

### [184. 部门工资最高的员工](https://leetcode-cn.com/problems/department-highest-salary/)

```mysql
-- SQL架构
Create table If Not Exists Employee (Id int, Name varchar(255), Salary int, DepartmentId int);
Create table If Not Exists Department (Id int, Name varchar(255));
Truncate table Employee;
insert into Employee (Id, Name, Salary, DepartmentId) values ('1', 'Joe', '70000', '1');
insert into Employee (Id, Name, Salary, DepartmentId) values ('2', 'Jim', '90000', '1');
insert into Employee (Id, Name, Salary, DepartmentId) values ('3', 'Henry', '80000', '2');
insert into Employee (Id, Name, Salary, DepartmentId) values ('4', 'Sam', '60000', '2');
insert into Employee (Id, Name, Salary, DepartmentId) values ('5', 'Max', '90000', '1');
Truncate table Department;
insert into Department (Id, Name) values ('1', 'IT');
insert into Department (Id, Name) values ('2', 'Sales');

-- Employee表包含所有员工信息，每个员工有其对应的Id, salary和department Id
+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 70000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
+----+-------+--------+--------------+

-- Department 表包含公司所有部门的信息
+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+

-- 编写一个SQL查询，找出每个部门工资最高的员工。例如，根据上述给定的表格，Max在IT部门有最高工资，Henry在Sales部门有最高工资
+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| Sales      | Henry    | 80000  |
+------------+----------+--------+

-- SQL-1
SELECT d.Name AS Department, e1.Name Employee, e1.Salary Salary
FROM Employee e1 JOIN Department d ON e1.DepartmentId = d.Id
WHERE Salary=(SELECT MAX(Salary) FROM Employee e2 WHERE e1.DepartmentId=e2.DepartmentId);

-- SQL-2
SELECT
	Department.NAME AS Department,
	Employee.NAME AS Employee,
	Salary 
FROM
	Employee,
	Department 
WHERE
	Employee.DepartmentId = Department.Id 
	AND ( Employee.DepartmentId, Salary ) 
    IN (SELECT DepartmentId, max( Salary ) 
        FROM Employee 
        GROUP BY DepartmentId );

-- SQL-3
SELECT
	Department.NAME AS Department,
	Employee.NAME AS Employee,
	Salary 
FROM
	Employee JOIN Department ON Employee.DepartmentId = Department.Id
WHERE 
	(Employee.DepartmentId, Salary) 
    IN (SELECT DepartmentId, MAX(Salary) 
        FROM Employee
        GROUP BY DepartmentId);
```

### [511. 游戏玩法分析 I](https://leetcode-cn.com/problems/game-play-analysis-i/)

```mysql
-- SQL架构
Create table If Not Exists Activity (player_id int, device_id int, event_date date, games_played int);
Truncate table Activity;
insert into Activity (player_id, device_id, event_date, games_played) values ('1', '2', '2016-03-01', '5');
insert into Activity (player_id, device_id, event_date, games_played) values ('1', '2', '2016-05-02', '6');
insert into Activity (player_id, device_id, event_date, games_played) values ('2', '3', '2017-06-25', '1');
insert into Activity (player_id, device_id, event_date, games_played) values ('3', '1', '2016-03-02', '0');
insert into Activity (player_id, device_id, event_date, games_played) values ('3', '4', '2018-07-03', '5');

-- 活动表Activity
-- 表的主键是 (player_id, event_date)。这张表展示了一些游戏玩家在游戏平台上的行为活动。每行数据记录了一名玩家在退出平台之前，当天使用同一台设备登录平台后打开的游戏的数目（可能是 0 个）。
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| player_id    | int     |
| device_id    | int     |
| event_date   | date    |
| games_played | int     |
+--------------+---------+

-- 写一条 SQL 查询语句获取每位玩家 第一次登陆平台的日期。查询结果的格式如下所示：
Activity 表：
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-05-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |
+-----------+-----------+------------+--------------+

Result 表：
+-----------+-------------+
| player_id | first_login |
+-----------+-------------+
| 1         | 2016-03-01  |
| 2         | 2017-06-25  |
| 3         | 2016-03-02  |
+-----------+-------------+

-- SQL-1
SELECT player_id, event_date AS first_login
FROM activity
WHERE (player_id, event_date)
IN (SELECT player_id, MIN(event_date) FROM activity GROUP BY player_id);

-- SQL-2
SELECT DISTINCT player_id, MIN(event_date) AS first_login
FROM Activity
GROUP BY player_id;
```

### [512. 游戏玩法分析 II](https://leetcode-cn.com/problems/game-play-analysis-ii/)

```mysql
-- SQL架构
Create table If Not Exists Activity (player_id int, device_id int, event_date date, games_played int);
Truncate table Activity;
insert into Activity (player_id, device_id, event_date, games_played) values ('1', '2', '2016-03-01', '5');
insert into Activity (player_id, device_id, event_date, games_played) values ('1', '2', '2016-05-02', '6');
insert into Activity (player_id, device_id, event_date, games_played) values ('2', '3', '2017-06-25', '1');
insert into Activity (player_id, device_id, event_date, games_played) values ('3', '1', '2016-03-02', '0');
insert into Activity (player_id, device_id, event_date, games_played) values ('3', '4', '2018-07-03', '5');

-- 活动表Activity
-- 表的主键是 (player_id, event_date)。这张表展示了一些游戏玩家在游戏平台上的行为活动。每行数据记录了一名玩家在退出平台之前，当天使用同一台设备登录平台后打开的游戏的数目（可能是 0 个）。
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| player_id    | int     |
| device_id    | int     |
| event_date   | date    |
| games_played | int     |
+--------------+---------+

-- 请编写一个 SQL 查询，描述每一个玩家首次登陆的设备名称。查询结果格式在以下示例中：
Activity 表：
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-05-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |
+-----------+-----------+------------+--------------+

Result table:
+-----------+-----------+
| player_id | device_id |
+-----------+-----------+
| 1         | 2         |
| 2         | 3         |
| 3         | 1         |
+-----------+-----------+

-- SQL
SELECT
	player_id, device_id
FROM
	Activity
WHERE
	(player_id, event_date) 
IN 
	(SELECT player_id, MIN(event_date) FROM Activity GROUP BY player_id);
```

### [534. 游戏玩法分析 III](https://leetcode-cn.com/problems/game-play-analysis-iii/)

```mysql
-- SQL架构
Create table If Not Exists Activity (player_id int, device_id int, event_date date, games_played int);
Truncate table Activity;
insert into Activity (player_id, device_id, event_date, games_played) values ('1', '2', '2016-03-01', '5');
insert into Activity (player_id, device_id, event_date, games_played) values ('1', '2', '2016-05-02', '6');
insert into Activity (player_id, device_id, event_date, games_played) values ('1', '3', '2017-06-25', '1');
insert into Activity (player_id, device_id, event_date, games_played) values ('3', '1', '2016-03-02', '0');
insert into Activity (player_id, device_id, event_date, games_played) values ('3', '4', '2018-07-03', '5');

-- 活动表Activity
-- 表的主键是 (player_id, event_date)。这张表展示了一些游戏玩家在游戏平台上的行为活动。每行数据记录了一名玩家在退出平台之前，当天使用同一台设备登录平台后打开的游戏的数目（可能是 0 个）。
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| player_id    | int     |
| device_id    | int     |
| event_date   | date    |
| games_played | int     |
+--------------+---------+

-- 编写一个 SQL 查询，同时报告每组玩家和日期，以及玩家到目前为止玩了多少游戏。也就是说，在此日期之前玩家所玩的游戏总数。详细情况请查看示例。查询结果格式如下所示：
Activity table:
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-05-02 | 6            |
| 1         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |
+-----------+-----------+------------+--------------+

Result table:
+-----------+------------+---------------------+
| player_id | event_date | games_played_so_far |
+-----------+------------+---------------------+
| 1         | 2016-03-01 | 5                   |
| 1         | 2016-05-02 | 11                  |
| 1         | 2017-06-25 | 12                  |
| 3         | 2016-03-02 | 0                   |
| 3         | 2018-07-03 | 5                   |
+-----------+------------+---------------------+
对于 ID 为 1 的玩家，2016-05-02 共玩了 5+6=11 个游戏，2017-06-25 共玩了 5+6+1=12 个游戏。
对于 ID 为 3 的玩家，2018-07-03 共玩了 0+5=5 个游戏。
请注意，对于每个玩家，我们只关心玩家的登录日期。

-- SQL
SELECT 
    player_id, 
    event_date, 
    CASE WHEN @prev = player_id THEN @cnt := @cnt + games_played 
         WHEN @prev := player_id THEN @cnt := games_played END 'games_played_so_far'
FROM 
    (SELECT player_id, event_date, games_played FROM activity ORDER BY player_id, event_date) a, 
    (SELECT @cnt := 0, @prev := null) t;
```

### [550. 游戏玩法分析 IV](https://leetcode-cn.com/problems/game-play-analysis-iv/)

```mysql
-- SQL架构
Create table If Not Exists Activity (player_id int, device_id int, event_date date, games_played int);
Truncate table Activity;
insert into Activity (player_id, device_id, event_date, games_played) values ('1', '2', '2016-03-01', '5');
insert into Activity (player_id, device_id, event_date, games_played) values ('1', '2', '2016-03-02', '6');
insert into Activity (player_id, device_id, event_date, games_played) values ('2', '3', '2017-06-25', '1');
insert into Activity (player_id, device_id, event_date, games_played) values ('3', '1', '2016-03-02', '0');
insert into Activity (player_id, device_id, event_date, games_played) values ('3', '4', '2018-07-03', '5');

-- 活动表Activity
-- 表的主键是 (player_id, event_date)。这张表展示了一些游戏玩家在游戏平台上的行为活动。每行数据记录了一名玩家在退出平台之前，当天使用同一台设备登录平台后打开的游戏的数目（可能是 0 个）。
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| player_id    | int     |
| device_id    | int     |
| event_date   | date    |
| games_played | int     |
+--------------+---------+

-- 编写一个 SQL 查询，报告在首次登录的第二天再次登录的玩家的分数，四舍五入到小数点后两位。换句话说，您需要计算从首次登录日期开始至少连续两天登录的玩家的数量，然后除以玩家总数。查询结果格式如下所示：
Activity table:
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-03-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |
+-----------+-----------+------------+--------------+

Result table:
+-----------+
| fraction  |
+-----------+
| 0.33      |
+-----------+
只有 ID 为 1 的玩家在第一天登录后才重新登录，所以答案是 1/3 = 0.33

-- SQL
SELECT ROUND(
    (SELECT 
     	COUNT(a1.player_id)
	FROM
    	activity a1 JOIN activity a2 ON DATEDIFF(a2.event_date, a1.event_date) = 1 AND a1.player_id = a2.player_id
    WHERE
     	(a1.player_id, a1.event_date) 
    IN 
     	(SELECT
         	DISTINCT player_id, MIN(event_date)
         FROM
         	activity
         GROUP BY
         	player_id)) / 
	(SELECT
     	COUNT(DISTINCT player_id)
     FROM
     	activity)
, 2) AS fraction;
```

### [570. 至少有5名直接下属的经理](https://leetcode-cn.com/problems/managers-with-at-least-5-direct-reports/)

```mysql
-- SQL架构
Create table If Not Exists Employee (Id int, Name varchar(255), Department varchar(255), ManagerId int);
Truncate table Employee;
insert into Employee (Id, Name, Department, ManagerId) values ('101', 'John', 'A', NULL);
insert into Employee (Id, Name, Department, ManagerId) values ('102', 'Dan', 'A', '101');
insert into Employee (Id, Name, Department, ManagerId) values ('103', 'James', 'A', '101');
insert into Employee (Id, Name, Department, ManagerId) values ('104', 'Amy', 'A', '101');
insert into Employee (Id, Name, Department, ManagerId) values ('105', 'Anne', 'A', '101');
insert into Employee (Id, Name, Department, ManagerId) values ('106', 'Ron', 'B', '101');

-- Employee表包含所有员工和他们的经理。每个员工都有一个Id，并且还有一列是经理的Id
+------+----------+-----------+----------+
|Id    |Name 	  |Department |ManagerId |
+------+----------+-----------+----------+
|101   |John 	  |A 	      |null      |
|102   |Dan 	  |A 	      |101       |
|103   |James 	  |A 	      |101       |
|104   |Amy 	  |A 	      |101       |
|105   |Anne 	  |A 	      |101       |
|106   |Ron 	  |B 	      |101       |
+------+----------+-----------+----------+

-- 给定Employee表，请编写一个SQL查询来查找至少有5名直接下属的经理。对于上表，您的SQL查询应该返回：
-- 注意:没有人是自己的下属。
+-------+
| Name  |
+-------+
| John  |
+-------+

-- SQL
SELECT
    Name
FROM 
    employee
WHERE 
    Id
IN
    (SELECT 
        ManagerId
    FROM
        employee
    WHERE
        ManagerId IS NOT NULL
    GROUP BY
        ManagerId
    HAVING
        COUNT(ManagerId) >= 5);
```

### [574. 当选者](https://leetcode-cn.com/problems/winning-candidate/)

```mysql
-- SQL架构
Create table If Not Exists Candidate (id int, Name varchar(255));
Create table If Not Exists Vote (id int, CandidateId int);
Truncate table Candidate;
insert into Candidate (id, Name) values ('1', 'A');
insert into Candidate (id, Name) values ('2', 'B');
insert into Candidate (id, Name) values ('3', 'C');
insert into Candidate (id, Name) values ('4', 'D');
insert into Candidate (id, Name) values ('5', 'E');
Truncate table Vote;
insert into Vote (id, CandidateId) values ('1', '2');
insert into Vote (id, CandidateId) values ('2', '4');
insert into Vote (id, CandidateId) values ('3', '3');
insert into Vote (id, CandidateId) values ('4', '2');
insert into Vote (id, CandidateId) values ('5', '5');

-- 表: Candidate
+-----+---------+
| id  | Name    |
+-----+---------+
| 1   | A       |
| 2   | B       |
| 3   | C       |
| 4   | D       |
| 5   | E       |
+-----+---------+  

-- 表: Vote
+-----+--------------+
| id  | CandidateId  |
+-----+--------------+
| 1   |     2        |
| 2   |     4        |
| 3   |     3        |
| 4   |     2        |
| 5   |     5        |
+-----+--------------+
id 是自动递增的主键，
CandidateId 是 Candidate 表中的 id.

-- 请编写sql语句来找到当选者的名字，上面的例子将返回当选者B.
+------+
| Name |
+------+
| B    |
+------+

-- 注意:你可以假设没有平局，换言之，最多只有一位当选者。

-- SQL
SELECT Name
FROM candidate
WHERE id = (SELECT CandidateId
            FROM (SELECT CandidateId, COUNT(CandidateId) count 
                  FROM vote
                  GROUP BY CandidateId
                  ORDER BY count DESC LIMIT 0, 1) t);
```

### [577. 员工奖金](https://leetcode-cn.com/problems/employee-bonus/)

```mysql
-- SQL架构
Create table If Not Exists Employee (EmpId int, Name varchar(255), Supervisor int, Salary int);
Create table If Not Exists Bonus (EmpId int, Bonus int);
Truncate table Employee;
insert into Employee (EmpId, Name, Supervisor, Salary) values ('3', 'Brad', NULL, '4000');
insert into Employee (EmpId, Name, Supervisor, Salary) values ('1', 'John', '3', '1000');
insert into Employee (EmpId, Name, Supervisor, Salary) values ('2', 'Dan', '3', '2000');
insert into Employee (EmpId, Name, Supervisor, Salary) values ('4', 'Thomas', '3', '4000');
Truncate table Bonus;
insert into Bonus (EmpId, Bonus) values ('2', '500');
insert into Bonus (EmpId, Bonus) values ('4', '2000');

-- 选出所有 bonus < 1000 的员工的 name 及其 bonus。

Employee 表单
+-------+--------+-----------+--------+
| empId |  name  | supervisor| salary |
+-------+--------+-----------+--------+
|   1   | John   |  3        | 1000   |
|   2   | Dan    |  3        | 2000   |
|   3   | Brad   |  null     | 4000   |
|   4   | Thomas |  3        | 4000   |
+-------+--------+-----------+--------+
empId 是这张表单的主关键字

Bonus 表单
+-------+-------+
| empId | bonus |
+-------+-------+
| 2     | 500   |
| 4     | 2000  |
+-------+-------+
empId 是这张表单的主关键字

-- 输出示例：
+-------+-------+
| name  | bonus |
+-------+-------+
| John  | null  |
| Dan   | 500   |
| Brad  | null  |
+-------+-------+

-- SQL
SELECT e.Name name, b.bonus bonus
FROM employee e LEFT JOIN bonus b ON e.empId = b.empId
WHERE b.bonus < 1000 OR b.bonus IS NULL;
```

### [578. 查询回答率最高的问题](https://leetcode-cn.com/problems/get-highest-answer-rate-question/)

```mysql
-- SQL架构
Create table If Not Exists survey_log (id int, action varchar(255), question_id int, answer_id int, q_num int, timestamp int);
Truncate table survey_log;
insert into survey_log (id, action, question_id, answer_id, q_num, timestamp) values ('5', 'show', '285', NULL, '1', '123');
insert into survey_log (id, action, question_id, answer_id, q_num, timestamp) values ('5', 'answer', '285', '124124', '1', '124');
insert into survey_log (id, action, question_id, answer_id, q_num, timestamp) values ('5', 'show', '369', NULL, '2', '125');
insert into survey_log (id, action, question_id, answer_id, q_num, timestamp) values ('5', 'skip', '369', NULL, '2', '126');

-- 从 survey_log 表中获得回答率最高的问题，survey_log 表包含这些列：uid, action, question_id, answer_id, q_num, timestamp。
-- uid 表示用户 id；action 有以下几种值："show"，"answer"，"skip"；当 action 值为 "answer" 时 answer_id 非空，而 action 值为 "show" 或者 "skip" 时 answer_id 为空；q_num 表示当前会话中问题的编号。请编写SQL查询来找到具有最高回答率的问题。

示例:
输入:
+------+-----------+--------------+------------+-----------+------------+
| uid  | action    | question_id  | answer_id  | q_num     | timestamp  |
+------+-----------+--------------+------------+-----------+------------+
| 5    | show      | 285          | null       | 1         | 123        |
| 5    | answer    | 285          | 124124     | 1         | 124        |
| 5    | show      | 369          | null       | 2         | 125        |
| 5    | skip      | 369          | null       | 2         | 126        |
+------+-----------+--------------+------------+-----------+------------+
输出:
+-------------+
| survey_log  |
+-------------+
|    285      |
+-------------+
-- 解释:问题285的回答率为 1/1，而问题369回答率为 0/1，因此输出285。
-- 注意: 回答率最高的含义是：同一问题编号中回答数占显示数的比例。

-- SQL
-- 题目理解：对于每道题来说，不管被回答多少次，题目数量就是1，也就是分母是1， 那么将题目转化为求每道题被回答的次数，也就是answer_id不为NULL的次数
SELECT question_id survey_log FROM survey_log 
WHERE answer_id IS NOT NULL
GROUP BY question_id
ORDER BY COUNT(answer_id) DESC
LIMIT 1;
```

### [580. 统计各专业学生人数](https://leetcode-cn.com/problems/count-student-number-in-departments/)

```mysql
-- SQL架构
CREATE TABLE IF NOT EXISTS student (student_id INT,student_name VARCHAR(45), gender VARCHAR(6), dept_id INT);
CREATE TABLE IF NOT EXISTS department (dept_id INT, dept_name VARCHAR(255));
Truncate table student;
insert into student (student_id, student_name, gender, dept_id) values ('1', 'Jack', 'M', '1');
insert into student (student_id, student_name, gender, dept_id) values ('2', 'Jane', 'F', '1');
insert into student (student_id, student_name, gender, dept_id) values ('3', 'Mark', 'M', '2');
Truncate table department;
insert into department (dept_id, dept_name) values ('1', 'Engineering');
insert into department (dept_id, dept_name) values ('2', 'Science');
insert into department (dept_id, dept_name) values ('3', 'Law');

-- 一所大学有 2 个数据表，分别是 student 和 department ，这两个表保存着每个专业的学生数据和院系数据。写一个查询语句，查询 department 表中每个专业的学生人数 （即使没有学生的专业也需列出）。将你的查询结果按照学生人数降序排列。 如果有两个或两个以上专业有相同的学生数目，将这些部门按照部门名字的字典序从小到大排列。

student 表格如下：

| Column Name  | Type      |
|--------------|-----------|
| student_id   | Integer   |
| student_name | String    |
| gender       | Character |
| dept_id      | Integer   |
其中， student_id 是学生的学号， student_name 是学生的姓名， gender 是学生的性别， dept_id 是学生所属专业的专业编号。

department 表格如下：

| Column Name | Type    |
|-------------|---------|
| dept_id     | Integer |
| dept_name   | String  |
dept_id 是专业编号， dept_name 是专业名字。

这里是一个示例输入：
student 表格：

| student_id | student_name | gender | dept_id |
|------------|--------------|--------|---------|
| 1          | Jack         | M      | 1       |
| 2          | Jane         | F      | 1       |
| 3          | Mark         | M      | 2       |
department 表格：

| dept_id | dept_name   |
|---------|-------------|
| 1       | Engineering |
| 2       | Science     |
| 3       | Law         |
示例输出为：

| dept_name   | student_number |
|-------------|----------------|
| Engineering | 2              |
| Science     | 1              |
| Law         | 0              |

-- SQL
SELECT d.dept_name, COUNT(student_id) AS student_number
FROM department d LEFT JOIN student s ON s.dept_id = d.dept_id
GROUP BY dept_name
ORDER BY student_number DESC, d.dept_name;
```

