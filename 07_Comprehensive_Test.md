# 07 Comprehensive Test

## 练习一: 各部门工资最高的员工（难度：中等）

```

创建Employee 表，包含所有员工信息，每个员工有其对应的 Id, salary 和 department Id。

+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 70000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
+----+-------+--------+--------------+
创建Department 表，包含公司所有部门的信息。

+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+
编写一个 SQL 查询，找出每个部门工资最高的员工。例如，根据上述给定的表格，Max 在 IT 部门有最高工资，Henry 在 Sales 部门有最高工资。

+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| Sales      | Henry    | 80000  |
+------------+----------+--------+
```

```mysql
-- 练习一
-- Answer:
CREATE DATABASE test;
USE test;
-- 创建Employee表
CREATE TABLE Employee(
Id INTEGER NOT NULL,
Name VARCHAR(20) NOT NULL,
Salary INTEGER NOT NULL,
DepartmentId INTEGER NOT NULL,
PRIMARY KEY (Id)
);
-- 向Employee表写入数据
INSERT INTO Employee VALUES(1, 'Joe', 70000, 1);
INSERT INTO Employee VALUES(2, 'Henry', 80000, 2);
INSERT INTO Employee VALUES(3, 'Sam', 60000, 2);
INSERT INTO Employee VALUES(4, 'Max', 90000, 1);

-- 创建Department表
CREATE TABLE Department(
Id INTEGER NOT NULL,
Name VARCHAR(10) NOT NULL,
PRIMARY KEY (Id)
);
-- 向Department表写入数据
INSERT INTO Department VALUES(1, 'IT');
INSERT INTO Department VALUES(2, 'Sales');

-- Query查询
-- Solution1：
-- 首先将两表进行连结
-- 巧妙利用MAX函数可进行字符串使用的特性，直接对连结后的表进行GROUP BY即可
-- 成也MAX，败也MAX，如果多个最高工资，则会出问题
SELECT MAX(D.Name) AS 'Department', MAX(E.Name) AS 'Employee', MAX(E.Salary) AS 'Salary'
FROM Employee AS E
LEFT JOIN Department AS D ON E.DepartmentId = D.Id
GROUP BY E.DepartmentId;

-- Solution2：
-- 首先将两表进行内连结
-- 使用(DepartmentId,MAX(Salary))合并子查询建立->部门+最高薪酬,然后使用IN筛选
-- IN谓语 可以轻松应对多个最高工资的情况，通用而实在，推荐。
SELECT D.Name AS 'Department', E.Name AS 'Employee', E.Salary
FROM Employee AS E INNER JOIN Department AS D ON E.DepartmentId = D.Id
WHERE (E.DepartmentId,E.Salary) IN (SELECT DepartmentId, MAX(Salary)
                  FROM Employee
                  GROUP BY DepartmentId);

```



## 练习二: 换座位（难度：中等）

```
小美是一所中学的信息科技老师，她有一张 seat 座位表，平时用来储存学生名字和与他们相对应的座位 id。

其中纵列的id是连续递增的

小美想改变相邻俩学生的座位。

你能不能帮她写一个 SQL query 来输出小美想要的结果呢？

请创建如下所示seat表：

示例：

+---------+---------+
|    id   | student |
+---------+---------+
|    1    | Abbot   |
|    2    | Doris   |
|    3    | Emerson |
|    4    | Green   |
|    5    | Jeames  |
+---------+---------+
假如数据输入的是上表，则输出结果如下：

+---------+---------+
|    id   | student |
+---------+---------+
|    1    | Doris   |
|    2    | Abbot   |
|    3    | Green   |
|    4    | Emerson |
|    5    | Jeames  |
+---------+---------+
注意：
如果学生人数是奇数，则不需要改变最后一个同学的座位。
```

```mysql
-- Answer:
CREATE TABLE seat(
 id INTEGER NOT NULL,
 student VARCHAR(20) NOT NULL,
 PRIMARY KEY (id)
 );
 
INSERT INTO seat VALUES(1, 'Abbot');
INSERT INTO seat VALUES(2, 'Doris');
INSERT INTO seat VALUES(3, 'Emerson');
INSERT INTO seat VALUES(4, 'Green');
INSERT INTO seat VALUES(5, 'Jeames');


```

