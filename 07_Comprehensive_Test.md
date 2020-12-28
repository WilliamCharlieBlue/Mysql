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

-- 解题思路：id为偶数时减一，奇数时加一，最后的id为奇数时不变

-- Solution1: 使用CASE实现
SELECT (CASE
    	-- 最大id是奇数的情况下，id不变
    	WHEN S.id%2 = 1 AND S.id = S1.largest THEN S.id
    	-- 奇数id加一
        -- 因为是CASE，而不是IF判断，因此不会出现最大id再加一的情况
    	WHEN S.id%2 = 1 THEN S.id + 1
    	-- 偶数id减一
    	ELSE S.id - 1
    	END) AS id, S.student
-- 从FROM子句中就可以把最大id计算出来了
FROM seat AS S, (SELECT COUNT(id) AS largest FROM seat) AS S1
ORDER BY id ASC;

-- Solution2: 也可以使用IF实现
-- IF(判断条件，真时返回值，假时返回值)
-- 这里的第一个IF的假时返回值嵌套了另一个IF
SELECT (
    	IF
        ( S.id%2 = 0, S.id-1, IF(S.id = S1.largest, S.id, S.id+1))
       ) AS id, S.student
FROM seat AS S, (SELECT COUNT(id) AS largest FROM seat) AS S1
ORDER BY id ASC;

```



## 练习三: 分数排名（难度：中等）

```
编写一个 SQL 查询来实现分数排名。如果两个分数相同，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有“间隔”。

创建以下score表：

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
例如，根据上述给定的 Scores 表，你的查询应该返回（按分数从高到低排列）：

+-------+------+
| Score | Rank |
+-------+------+
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |
+-------+------+
```

```mysql
-- Answer:
-- float(4,2) 表示：这个浮点数最大长度为4，也就是四位，小数部分为2位

CREATE TABLE score (
 id INTEGER NOT NULL,
 Score FLOAT(4,2) NOT NULL,
 PRIMARY KEY (id)
 );
 
INSERT INTO score VALUES(1, 3.50);
INSERT INTO score VALUES(2, 3.65);
INSERT INTO score VALUES(3, 4.00);
INSERT INTO score VALUES(4, 3.85);
INSERT INTO score VALUES(5, 4.00);
INSERT INTO score VALUES(6, 3.65);

-- 使用窗口专用函数DENSE_RANK,重复值同序，非重复值连续
-- 在OVER条件中，使用降序排列
SELECT Score, DENSE_RANK() OVER (ORDER BY Score DESC) AS 'RANK'
FROM score;

```



## 练习四：连续出现的数字（难度：中等）

```
编写一个 SQL 查询，查找所有至少连续出现三次的数字。

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
例如，给定上面的 Logs 表， 1 是唯一连续出现至少三次的数字。

+-----------------+
| ConsecutiveNums |
+-----------------+
| 1               |
+-----------------+
```

```mysql
-- Answer:
CREATE TABLE number (
 Id INTEGER NOT NULL,
 Num INTEGER NOT NULL,
 PRIMARY KEY (id)
 );
 
INSERT INTO number VALUES(1, 1);
INSERT INTO number VALUES(2, 1);
INSERT INTO number VALUES(3, 1);
INSERT INTO number VALUES(4, 2);
INSERT INTO number VALUES(5, 1);
INSERT INTO number VALUES(6, 2);
INSERT INTO number VALUES(7, 2);

-- Solution1: 使用自连结，简单，但复杂度大，扩展性差
-- 这里使用的是过时的连结语法，但本质一样，在自连结使用时，结构更清晰。
-- 列出三个表格，第一个从ID1开始，第二个从ID2开始，第三个从ID3开始
SELECT DISTINCT N1.Num AS ConsecutiveNums
FROM number AS N1, number AS N2, number AS N3
WHERE N2.Id = N1.Id +1 AND N3.Id = N2.Id + 1
	  AND N2.Num = N1.Num AND N3.Num = N2.Num;

-- Solution2: 自定义变量进行条件判断，只不过本课程好像没涉及到,较标准的函数思维
-- 速度较Solution1快，且可以设置任何连续次数，用CNT来判断
SELECT DISTINCT N.Num AS ConsecutiveNums, N.CNT
FROM
(SELECT Num,
 	(CASE
 		-- 如果和之前的值相等，@count变量加一
 		WHEN @prev = Num THEN @count := @count+1
 		-- 如果不相等，@count重新赋值为1
 		ELSE (@prev := Num) and (@count := 1)
 		END ) AS CNT
 		-- 初始化是0，进入CASE后，第一个如果不是0，@count将赋值为1
 	FROM number, (SELECT @prev := 0, @count := 0) AS T
	
) AS N
WHERE N.CNT >=3;

-- 本题若是连续次数为2，可如下设置，结果如下表
-- WHERE N.CNT >=2;
+-----------------+------+
| ConsecutiveNums | CNT  |
+-----------------+------+
|               1 |    2 |
|               1 |    3 |
|               2 |    2 |
+-----------------+------+


-- Solution3: 窗口函数，LAG
-- 首先对Num进行了PARTITION BY, 对ID进行了ORDER BY
-- LAG中offset是往前返回第offset行的值
-- 如果没有第offset行，返回default_value；如果没有default_value返回NULL
LAG(<expression>[,offset[, default_value]]) OVER (
    PARTITION BY expr,...
    ORDER BY expr [ASC|DESC],...
) 
-- 虽然看起来高端，但是有点费解

SELECT DISTINCT N.Num AS ConsecutiveNums, N.id, N.prev
FROM
(	SELECT Num, Id, 
 		LAG(Id, 2) OVER (PARTITION BY Num ORDER BY Id) AS prev
 	FROM number
 ) AS N
 WHERE N.ID = N.prev + 2;

-- 不指定WHERE和SELCET条件时是这样的情况
+-----+----+------+
| Num | Id | prev |
+-----+----+------+
|   1 |  1 | NULL |
|   1 |  2 | NULL |
|   1 |  3 |    1 |
|   1 |  5 |    2 |
|   2 |  4 | NULL |
|   2 |  6 | NULL |
|   2 |  7 |    4 |
+-----+----+------+
```



## 练习五：树节点 （难度：中等）

```
对于tree表，id是树节点的标识，p_id是其父节点的id。

+----+------+
| id | p_id |
+----+------+
| 1  | null |
| 2  | 1    |
| 3  | 1    |
| 4  | 2    |
| 5  | 2    |
+----+------+
每个节点都是以下三种类型中的一种：

Root: 如果节点是根节点。
Leaf: 如果节点是叶子节点。
Inner: 如果节点既不是根节点也不是叶子节点。
写一条查询语句打印节点id及对应的节点类型。按照节点id排序。上面例子的对应结果为：

+----+------+
| id | Type |
+----+------+
| 1  | Root |
| 2  | Inner|
| 3  | Leaf |
| 4  | Leaf |
| 5  | Leaf |
+----+------+
说明

节点’1’是根节点，因为它的父节点为NULL，有’2’和’3’两个子节点。
节点’2’是内部节点，因为它的父节点是’1’，有子节点’4’和’5’。
节点’3’，‘4’，'5’是叶子节点，因为它们有父节点但没有子节点。
下面是树的图形：

    1         
  /   \ 
 2    3    
/ \
4  5
注意

如果一个树只有一个节点，只需要输出根节点属性。
```

```mysql
-- Answer:
CREATE TABLE tree (
 id INTEGER NOT NULL,
 p_id INTEGER ,
 PRIMARY KEY (id)
 );
 
INSERT INTO tree VALUES(1, NULL);
INSERT INTO tree VALUES(2, 1);
INSERT INTO tree VALUES(3, 1);
INSERT INTO tree VALUES(4, 2);
INSERT INTO tree VALUES(5, 2);

-- Solution1: CASE判断，较标准的函数思维
-- 结合NOT IN作为判断
SELECT id, 
	   (CASE
        WHEN (SELECT count(*) FROM tree) = 1 THEN 'Root'
        WHEN p_id is NULL THEN 'Root'
        WHEN id NOT IN (SELECT DISTINCT T1.id
                        FROM tree AS T1, tree AS T2
                        WHERE T1.id = T2.p_id) THEN 'Leaf'
        ELSE 'Inner'
        END) AS Type
FROM tree;




-- Solution2: 左连结
-- 核心在于T1.id = T2.p_id，用id去与p_id作为左连结条件
-- 如果一个节点没有子节点的话，T2.id 和 T2.p_id都为NULL，这就是第二层IF的依据
-- 同时由于IF中第一个判断的即是否为Root，因此不需要额外的判断了
SELECT DISTINCT T1.id, 
	IF(T1.p_id IS NULL, 'Root', 
       IF(T2.p_id IS NULL, 'Leaf', 'Inner')) AS 'Type'
FROM tree AS T1 LEFT JOIN tree AS T2 ON T1.id = T2.p_id;

-- 未指定IF 和 SELECT条件时是这样的
+----+------+------+------+
| id | p_id | id   | p_id |
+----+------+------+------+
|  1 | NULL |    3 |    1 |
|  1 | NULL |    2 |    1 |
|  2 |    1 |    5 |    2 |
|  2 |    1 |    4 |    2 |
|  3 |    1 | NULL | NULL |
|  4 |    2 | NULL | NULL |
|  5 |    2 | NULL | NULL |
+----+------+------+------+
```



## 练习六：至少有五名直接下属的经理 （难度：中等）

```
Employee表包含所有员工及其上级的信息。每位员工都有一个Id，并且还有一个对应主管的Id（ManagerId）。

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
针对Employee表，写一条SQL语句找出有5个下属的主管。对于上面的表，结果应输出：

+-------+
| Name  |
+-------+
| John  |
+-------+
注意:

没有人向自己汇报。
```

```mysql
-- Answer: 
CREATE TABLE employee(
Id INTEGER NOT NULL,
Name VARCHAR(10) NOT NULL,
Department CHAR(1) NOT NULL,
ManagerId INTEGER,
PRIMARY KEY (Id)
);

INSERT INTO employee VALUES(101, 'John', 'A', NULL);
INSERT INTO employee VALUES(102, 'Dan', 'A', 101);
INSERT INTO employee VALUES(103, 'James', 'A', 101);
INSERT INTO employee VALUES(104, 'Amy', 'A', 101);
INSERT INTO employee VALUES(105, 'Anne', 'A', 101);
INSERT INTO employee VALUES(106, 'Ron', 'B', 101);


-- Solution1 利用IN + 子查询 + HAVING
SELECT Name
FROM employee
WHERE Id IN (SELECT ManagerId
             FROM employee
             GROUP BY ManagerId
             HAVING (COUNT(*) >= 5)  		
			);

-- Solution2 左连结 + HAVING
SELECT E1.Name
SELECT *
FROM employee AS E1 LEFT JOIN employee AS E2 ON E1.Id = E2.ManagerId;
GROUP BY E1.Name
HAVING (COUNT(*) >= 5)  

-- Solution3 先筛选+ 联合查询 + WHERE
-- 可方便显示具体的个数
SELECT E1.Name, E2.num
FROM employee AS E1, 
	(SELECT ManagerId, COUNT(Id) AS num
     FROM employee 
     GROUP BY ManagerId
    ) AS E2
WHERE E1.ID = E2.ManagerId AND E2.num >=5;

-- 单纯E2的结果如下：
+-----------+-----------+
| ManagerId | COUNT(Id) |
+-----------+-----------+
|      NULL |         1 |
|       101 |         5 |
+-----------+-----------+
```



## 练习七: 分数排名 （难度：中等）

```
练习三的分数表，实现排名功能，但是排名需要是非连续的，如下：

+-------+------+
| Score | Rank |
+-------+------+
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 3    |
| 3.65  | 4    |
| 3.65  | 4    |
| 3.50  | 6    |
+-------+------
```

```mysql
-- Answer:
-- 使用窗口专用函数DENSE_RANK(),重复值同序，非重复值连续
-- 在OVER条件中，使用降序排列
-- SELECT Score, DENSE_RANK() OVER (ORDER BY Score DESC) AS 'RANK'
-- FROM score;

-- 使用窗口专用函数RANK(),重复值同序且占位，非重复值不连续
SELECT Score, RANK() OVER (ORDER BY Score DESC) AS 'RANK'
FROM score;
```



## 练习八：查询回答率最高的问题 （难度：中等）

```
求出survey_log表中回答率最高的问题，表格的字段有：uid, action, question_id, answer_id, q_num, timestamp。

uid是用户id；action的值为：“show”， “answer”， “skip”；当action是"answer"时，answer_id不为空，相反，当action是"show"和"skip"时为空（null）；q_num是问题的数字序号。

写一条sql语句找出回答率最高的问题。

举例：

输入

uid	action	question_id	answer_id	q_num	timestamp
5	show	285	null	1	123
5	answer	285	124124	1	124
5	show	369	null	2	125
5	skip	369	null	2	126
输出

survey_log
285
说明

问题285的回答率为1/1，然而问题369的回答率是0/1，所以输出是285。

**注意：**最高回答率的意思是：同一个问题出现的次数中回答的比例。
```

```mysql
-- Answer:
CREATE TABLE survey_log(
uid INTEGER NOT NULL,
action VARCHAR(10) NOT NULL,
question_id INTEGER NOT NULL,
answer_id INTEGER,
q_num INTEGER NOT NULL, 
timestamp INTEGER NOT NULL
);

INSERT INTO survey_log VALUES(5, 'show', 285, NULL, 1, 123);
INSERT INTO survey_log VALUES(5, 'answer', 285, 124124, 1, 124);
INSERT INTO survey_log VALUES(5, 'show', 369, NULL, 2, 125);
INSERT INTO survey_log VALUES(5, 'skip', 369, NULL, 2, 126);

-- 这个题目比较有意思，survey_log记录的到底是什么。
-- 该表记录的是每个选手答题的流程：
-- 题目出现（show）-回答（answer）/跳过（skip）
-- 下一题出现（show）-回答（answer）/跳过（skip）
-- 每个题目出现（show）以后，选手只能从回答（answer）或者跳过（skip）中选择一种对该题进行应对
-- 要么回答（answer），要么跳过（skip）。回答或跳过以后，再进入下一题。

-- show->出了几题；answer->答了几题

-- Solution1 在子查询中进行计算筛选
SELECT question_id AS 'survey_log'
FROM(
	SELECT question_id, 
    	(SUM(CASE WHEN 'action' like 'answer' THEN 1 ELSE 0 END)/SUM(CASE WHEN 'action' like 'show' THEN 1 ELSE 0 END)       
   		) AS rate
    FROM survey_log
    GROUP BY question_id
    ORDER BY rate DESC
    -- 这里只选择降序排列的第一个，即为回答率最高的
    LIMIT 1
) AS S;

-- Solution2 先统计再计算
SELECT question_id AS 'survey_log'
FROM(
    SELECT question_id,
		SUM(CASE WHEN 'action' like 'answer' THEN 1 ELSE 0 END) AS num_answer,
    	SUM(CASE WHEN 'action' like 'show' THEN 1 ELSE 0 END) AS num_show
    FROM survey_log
    GROUP BY question_id
) AS S
ORDER BY (num_answer/num_show) DESC
LIMIT 1;

-- Solution3 在排序中计算筛选，COUNT + IF
SELECT question_id AS 'survey_log'
FROM survey_log
GROUP BY question_id
ORDER BY COUNT(IF(action='answer', 1, 0))/COUNT(IF(action='show', 1, 0)) DESC
-- 可直接统计answer_id,将忽略answer_id为NULL的行
-- ORDER BY COUNT(answer_id))/COUNT(IF(action='show', 1, 0)) DESC
LIMIT 1;

```



## 练习九：各部门前3高工资的员工（难度：中等

```
将项目7中的employee表清空，重新插入以下数据（其实是多插入5,6两行）：

+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 70000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
| 5  | Janet | 69000  | 1            |
| 6  | Randy | 85000  | 1            |
+----+-------+--------+--------------+
编写一个 SQL 查询，找出每个部门工资前三高的员工。例如，根据上述给定的表格，查询结果应返回：

+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| IT         | Randy    | 85000  |
| IT         | Joe      | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |
+------------+----------+--------+
此外，请考虑实现各部门前N高工资的员工功能。
```

```mysql
-- Answer:
INSERT INTO Employee VALUES(5, 'Janet', 69000, 1);
INSERT INTO Employee VALUES(6, 'Randy', 85000, 1);

-- 使用窗口函数的PARTITION + ORDER DESC，获取各部门的倒序工资
-- 只要按各部门的ranking筛选前几就可以了
SELECT T.Department, T.Employee, T.Salary
FROM (SELECT D.Name AS 'Department', E.Name AS 'Employee', E.Salary AS 'Salary',
      RANK() OVER (PARTITION BY E.DepartmentId ORDER BY E.Salary DESC) AS ranking
      FROM Employee AS E INNER JOIN Department AS D ON E.DepartmentId = D.Id
) AS  T
WHERE T.ranking <=3;
-- 各部门前N高工资的员工功能
-- WHERE T.ranking <=n

```



## 练习十：平面上最近距离 (难度: 困难

```
point_2d表包含一个平面内一些点（超过两个）的坐标值（x，y）。

写一条查询语句求出这些点中的最短距离并保留2位小数。

|x   | y  |
|----|----|
| -1 | -1 |
|  0 |  0 |
| -1 | -2 |
最短距离是1，从点（-1，-1）到点（-1，-2）。所以输出结果为：

| shortest |

1.00

+--------+
|shortest|
+--------+
|1.00    |
+--------+
注意： 所有点的最大距离小于10000。
```

```mysql

```

