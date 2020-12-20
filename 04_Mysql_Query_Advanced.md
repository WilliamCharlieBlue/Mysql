# 04 Advanced Mysql Query 

## 4.1 View 视图

```mysql
-- 视图是依据SELECT语句来创建的一个虚拟的表
CREATE VIEW <视图名称>(<列名1>,<列名2>,...) AS <SELECT语句>
-- 视图的查询方式和正常查询方式一致
-- TABLE查询
SELECT stu_name FROM students_table;
-- VIEW查询
SELECT stu_name FROM view_students_info;
```

### 4.1.1 视图的意义

```
- 视图是基于真实表的一张虚拟表，数据来源均建立在真实表的基础上
- 视图的基础上可以继续创建视图（需尽量避免，一般情况下，多重视图会降低SQL的性能）
- 使用视图的优点
	- 将频繁使用的SELECT语句保存以提高效率。
	- 使用户看到的数据更加清晰。
	- 不用对外公开数据表全部字段，增强数据的保密性。
	- 降低数据的冗余。
```

### 4.1.2 视图的创建

```
CREATE VIEW <视图名称>(<列名1>,<列名2>,...) AS <SELECT语句>
- 视图名在数据库中要求唯一，不能与表和其他视图重名。
- SELECT语句写在AS关键字之后。
- SELECT 语句中列的排列顺序和视图中列的排列顺序相同。
- DBMS中定义视图时不能使用ORDER BY语句，MySQL中允许，尽量避免使用，多重视图中新建视图的ORDER BY可能会被忽略。
```

```mysql
-- 创建一个视图
CREATE VIEW productsum (product_type, cnt_product)
AS
SELECT product_type, COUNT(*)
  FROM product
 GROUP BY product_type ;
-- 查看视图 
 SELECT * FROM productsum;
```

### 4.1.3 多表视图的创建

```mysql
-- 从product, shop_product表里拿出产品id一样的条目，列出销售价格和店名作为视图
CREATE VIEW view_shop_product(product_type, sale_price, shop_name)
AS
SELECT product_type, sale_price, shop_name
  FROM product,
       shop_product
 WHERE product.product_id = shop_product.product_id;
 
-- 查看新建的视图
SELECT * FROM view_shop_product;

-- 对视图进行查询，商品为'衣服'的视图条目
SELECT sale_price, shop_name
  FROM view_shop_product
 WHERE product_type = '衣服';

```

### 4.1.4 修改视图结构和内容

```mysql
ALTER VIEW <视图名> AS <SELECT语句>
- 再次强调！视图名在数据库中是唯一的，不能与表和其他视图重名
- 最暴力的修改方式，是直接删除视图，然后重建创建视图

- 重点重点！我们在创建视图时尽量使用限制不允许通过视图来修改表。
- 视图内容改变，真实表的内容也会改变。
- 视图是一个虚拟表，所以对视图的操作就是对底层基础表的操作。
- 但更新视图内容也有条件，包含以下结构的任意一种都不能修改：
	- 聚合函数 SUM()、MIN()、MAX()、COUNT()等。
	- DISTINCT关键字。
	- GROUP BY子句。
	- HAVING子句。
	- UNION 或 UNION ALL运算符。
	- FROM子句中包含多个表。
```

```mysql
-- 修改视图结构
ALTER VIEW productsum
    AS
        SELECT product_type, sale_price
          FROM product
         WHERE regist_date > '2009-09-11';
         
-- 修改视图内容   
UPDATE productsum
   SET sale_price = '5000'
 WHERE product_type = '办公用品';
```

### 4.1.5 删除视图

```mysql
DROP VIEW <视图名1> [ , <视图名2> …]
- 删除需要权限
```

```mysql
DROP VIEW productsum;
-- 再次删除时，会提示操作内容不存在
DROP VIEW productsum;
```



## 4.2 SubQuery 子查询

### 4.2.1 子查询的意义

```mysql
SELECT stu_name
FROM (SELECT stu_name, COUNT(*) AS stu_cnt FROM students_info GROUP BY stu_age) 
AS studentSum;

- 子查询指一个查询语句嵌套在另一个查询语句内部的查询。
- 先执行FROM里的SQL语句，执行成功后，再执行外面的SQL语句。
- 查询可以基于一个表或者多个表。
- 子查询和视图的区别
	- 子查询的定义直接放在FROM子句中
	- 子查询是一次性的，不会保存，SELECT语句执行之后就消失
```

### 4.2.2 嵌套子查询

```mysql
-- 子查询中再嵌套一层子查询
-- 随着子查询嵌套的层数的叠加，SQL语句不仅会难以理解而且执行效率越来越差，尽量避免
SELECT product_type, cnt_product
FROM (  SELECT *
        FROM (  SELECT product_type, COUNT(*) AS cnt_product
                FROM product 
                GROUP BY product_type
             ) 
        AS productsum
        WHERE cnt_product = 4
      )
AS productsum2;
```

### 4.2.3 标量子查询

```mysql
-- 标量就是单一，执行的SQL语句只能返回一个值，即某一行的某一列
-- 标量子查询可以用在WHERE、SELECT、GROUP BY、HAVING、ORDER BY等子句中

-- WHERE中插入标量子查询语句，查询出销售单价高于平均销售单价的商品。
SELECT product_id, product_name, sale_price
  FROM product
 WHERE sale_price > (SELECT AVG(sale_price) FROM product);
 
 
-- SELECT中插入标量子查询语句，列出平均值
SELECT product_id, product_name, sale_price,
       (SELECT AVG(sale_price) FROM product) AS avg_price
FROM product;
```

### 4.2.4 关联子查询

```mysql
-- 同过关键词，将两个查询关联起来
-- 下面的分解为
	-- p1查询三列信息
	-- p2使用GROUP BY获取给商品种类平均价格
	-- 两次查询使用WHERE p1.product_type = p2.product_type关联在一起
SELECT product_type, product_name, sale_price
  FROM product AS p1
 WHERE sale_price > (SELECT AVG(sale_price)
                       FROM product AS p2
                      WHERE p1.product_type = p2.product_type
                      GROUP BY product_type);
```



## 4.3 Mysql常用函数

```
- 算术函数 （用来进行数值计算的函数）
- 字符串函数 （用来进行字符串操作的函数）
- 日期函数 （用来进行日期操作的函数）
- 转换函数 （用来转换数据类型和值的函数）
- 聚合函数 （用来进行数据聚合的函数）
```

### 4.3.1 算数函数

```mysql
-- 四则运算: + - * /
-- ABS() 绝对值
-- MOD() 求余数
-- ROUND() 四舍五入

SELECT m, ABS(m) AS abs_col, n, p, 
       MOD(n, p) AS mod_col, 
       ROUND(m,1) AS round_colS
FROM samplemath;
```

### 4.3.2 字符串函数

```mysql
-- CONCAT 拼接
CONCAT(str1, str2, str3)
-- LENGTH 字符串长度
LENGTH(str1)
-- UPPER/LOWER 大/小写转换，只能针对英文字母使用
UPPER(str1) AND LOWER(str2)
-- REPLACE 字符串的替换,把abc替换为def
REPLACE(str1, 'abc', 'def')
-- SUBSTRING 字符串的截取
	-- SUBSTRING （对象字符串 FROM 截取的起始位置 FOR 截取的字符数）
	-- 从字符串最左侧开始计算，索引值起始为1。
SUBSTRING(str1 FROM 3 FOR 2)
-- SUBSTRING_INDEX 字符串按索引截取
	-- SUBSTRING_INDEX (原始字符串， 分隔符，n)
	-- n表示第n个分隔符，而不是每个分隔符都分隔然后选索引
	-- 多个分隔符之间的字符串，需要多次拆分
	-- 按分隔符分割，支持正向和反向索引，索引起始值分别为 1 和 -1。
SELECT SUBSTRING_INDEX('www.mysql.com', '.', -2);
SELECT SUBSTRING_INDEX(SUBSTRING_INDEX('www.mysql.com', '.', -2), '.', 1);

SELECT
	str1,
	str2,
	str3,
	CONCAT(str1, str2, str3) AS str_concat,
	LENGTH(str1) AS len_str,
	LOWER(str1) AS low_str,
	REPLACE(str1, str2, str3) AS rep_str,
	SUBSTRING(str1 FROM 3 FOR 2) AS sub_str
FROM samplestr;
```

4.3.2 日期函数

```mysql
-- 不同DBMS的日期函数语法各有不同，以下为通用的一些函数
SELECT CURRENT_DATE; -- 获取当前日期
SELECT CURRENT_TIME; -- 当前时间
SELECT CURRENT_TIMESTAMP; -- 当前日期和时间 2020-12-20 17:43:28
-- EXTRACT 截取日期元素 
	-- EXTRACT(日期元素 FROM 日期)，可以是年/月/日/时/分/秒
SELECT CURRENT_TIMESTAMP as now,
EXTRACT(YEAR   FROM CURRENT_TIMESTAMP) AS year,
EXTRACT(MONTH  FROM CURRENT_TIMESTAMP) AS month,
EXTRACT(DAY    FROM CURRENT_TIMESTAMP) AS day,
EXTRACT(HOUR   FROM CURRENT_TIMESTAMP) AS hour,
EXTRACT(MINUTE FROM CURRENT_TIMESTAMP) AS MINute,
EXTRACT(SECOND FROM CURRENT_TIMESTAMP) AS second;
```

### 4.3.3 转换函数

```mysql
-- CAST 数据类型的转换
	-- CAST（转换前的值 AS 想要转换的数据类型）
	-- 将字符串类型转换为数值类型
SELECT CAST('0001' AS SIGNED INTEGER) AS int_col;
	-- 将字符串类型转换为日期类型
SELECT CAST('2009-12-14' AS DATE) AS date_col;

-- COALESCE 将NULL转换为其他值
	-- COALESCE(数据1，数据2，数据3……)
	-- 返回可变参数 A 中左侧开始第 1个不是NULL的值
SELECT COALESCE(NULL, 11) AS col_1,
       COALESCE(NULL, 'hello world', NULL) AS col_2,
       COALESCE(NULL, NULL, '2020-11-01') AS col_3;

```



## 4.4 Logical Operators 谓词

```
- 真值: TRUE / FALSE / UNKNOWN
- 谓词就是返回值为真值的函数
	- LIKE
	- BETWEEN
	- IS NULL、IS NOT NULL
	- IN
	- EXISTS
- 谓词的作用就是 “判断是否存在满足某种条件的记录”，如果存在这样的记录就返回真（TRUE），如果不存在就返回假（FALSE）。
```

### 4.4.1 LIKE谓词 – 用于字符串的部分一致查询

```mysql
-- 前方、中间、后方一致
-- 前方一致：选取出“dddabc”
SELECT * FROM samplelike WHERE strcol LIKE 'ddd%';
-- 中间一致：选取出“abcddd”, “dddabc”, “abdddc”
SELECT * FROM samplelike WHERE strcol LIKE '%ddd%';
-- 后方一致：选取出“abcddd”
SELECT * FROM samplelike WHERE strcol LIKE '%ddd';
```

### 4.4.2 BETWEEN谓词 – 用于范围查询

```mysql
-- 使用 BETWEEN 可以进行范围查询。该谓词与其他谓词或者函数的不同之处在于它使用了 3 个参数。
SELECT product_name, sale_price 
FROM product
WHERE sale_price BETWEEN 100 AND 1000;

-- BETWEEN 的特点就是结果中会包含 100 和 1000 这两个临界值，也就是闭区间。
SELECT product_name, sale_price
FROM product
WHERE sale_price > 100 AND sale_price < 1000;

```

### 4.4.3 IS NULL、 IS NOT NULL – 用于判断是否为NULL

```mysql
-- 为了选取出某些值为 NULL 的列的数据，不能使用 =，而只能使用特定的谓词IS NULL
SELECT product_name, purchase_price
FROM product
WHERE purchase_price IS NULL;
-- 想要选取 NULL 以外的数据时，需要使用IS NOT NULL。
SELECT product_name, purchase_price
FROM product
WHERE purchase_price IS NOT NULL;
```

### 4.4.3 IN谓词 – OR的简便用法

```mysql
-- 多个查询条件取并集时可以选择使用or语句。
SELECT product_name, purchase_price
FROM product
WHERE purchase_price = 320 OR purchase_price = 500 OR purchase_price = 5000;
-- IN
SELECT product_name, purchase_price
FROM product
WHERE purchase_price IN (320, 500, 5000);
-- NOT IN
SELECT product_name, purchase_price
FROM product
WHERE purchase_price NOT IN (320, 500, 5000);

-- IN谓语的NULL陷阱
	-- IN 和 NOT IN 时是无法选取出NULL数据的。
	-- NULL 只能使用 IS NULL 和 IS NOT NULL 来进行判断。

-- 使用子查询作为IN谓词的参数
	-- 子查询就是 SQL内部生成的表，因此也可以说“能够将表作为 IN 的参数”。
    -- NOT IN 与 IN用法一样
SELECT product_name, sale_price
FROM product
WHERE product_id IN 
	(SELECT product_id
  	 FROM shopproduct
     WHERE shop_id = '000C');
```

### 4.4.4 EXISTS 谓词

```mysql
-- 实际上即使不使用 EXISTS，基本上也都可以使用 IN（或者 NOT IN）来代替
-- 但熟练使用 EXISTS 谓词，就能体会到它极大的便利性。
-- EXISTS(记录)
-- 可以把在 EXISTS的子查询中书写 SELECT * 当作 SQL 的一种习惯。
-- NOT EXISTS 与 EXISTS用法一致

SELECT product_name, sale_price
  FROM product AS p
 WHERE EXISTS (SELECT *
               FROM shopproduct AS sp
               WHERE sp.shop_id = '000C'
               AND sp.product_id = p.product_id);
```



## 4.5 CASE 表达式

```
- CASE 表达式是在区分情况时使用的，通常成为(条件)分支
- 分为简单CASE表达式和搜索CASE表达式
- 依次判断 when 表达式是否为真值，是则执行 THEN 后的语句，如果所有的 when 表达式均为假，则执行 ELSE 后的语句。
- 最后只会返回一个值

CASE WHEN <求值表达式> THEN <表达式>
     WHEN <求值表达式> THEN <表达式>
     WHEN <求值表达式> THEN <表达式>
     .
ELSE <表达式>
END 

- ELSE子句可以忽略，会被默认为ELSE NULL。
- END千万不能省略，否则会出现语法错误。
```

### 4.5.1 CASE表达式的使用方法

```
- 场景一：取分支 (纯CASE WHEN 表达式)
	- 
- 场景二：行转列 (聚合函数 + CASE WHEN 表达式)
	- 当待转换列为数字时，可以使用SUM AVG MAX MIN等聚合函数；
	- 当待转换列为文本时，可以使用MAX MIN等聚合函数
```

```mysql
-- 应用场景1：根据不同分支得到不同列值
SELECT  product_name,
        CASE WHEN product_type = '衣服' THEN CONCAT('A ： ',product_type)
             WHEN product_type = '办公用品'  THEN CONCAT('B ： ',product_type)
             WHEN product_type = '厨房用具'  THEN CONCAT('C ： ',product_type)
             ELSE NULL
        END AS abc_product_type
FROM  product;

-- 应用场景2：实现列方向上的聚合
	-- 行方向上的聚合
SELECT product_type, SUM(sale_price) AS sum_price 
FROM product
GROUP BY product_type;  
	-- 聚合函数 + CASE WHEN 表达式 实现列方向的聚合
SELECT SUM(CASE WHEN product_type = '衣服' THEN sale_price ELSE 0 END) AS sum_price_clothes,
       SUM(CASE WHEN product_type = '厨房用具' THEN sale_price ELSE 0 END) AS sum_price_kitchen,
       SUM(CASE WHEN product_type = '办公用品' THEN sale_price ELSE 0 END) AS sum_price_office
FROM product;


-- 应用场景3：实现行转列
	-- 这里时数字列score实现行转列
SELECT name,
       SUM(CASE WHEN subject = '语文' THEN score ELSE null END) as chinese,
       SUM(CASE WHEN subject = '数学' THEN score ELSE null END) as math,
       SUM(CASE WHEN subject = '外语' THEN score ELSE null END) as english
FROM score
GROUP BY name;
	-- 也可以对文本列subject实现行转列 
SELECT name,
       MAX(CASE WHEN subject = '语文' THEN subject ELSE null END) as chinese,
       MAX(CASE WHEN subject = '数学' THEN subject ELSE null END) as math,
       MIN(CASE WHEN subject = '外语' THEN subject ELSE null END) as english
FROM score
GROUP BY name;
```



# Practice

```
3.1
创建出满足下述三个条件的视图（视图名称为 ViewPractice5_1）。使用 product（商品）表作为参照表，假设表中包含初始状态的 8 行数据。

条件 1：销售单价大于等于 1000 日元。
条件 2：登记日期是 2009 年 9 月 20 日。
条件 3：包含商品名称、销售单价和登记日期三列。
对该视图执行 SELECT 语句的结果如下所示。

SELECT * FROM ViewPractice5_1;
执行结果
product_name | sale_price | regist_date
--------------+------------+------------
T恤衫         |   1000    | 2009-09-20
菜刀          |    3000    | 2009-09-20
```

```mysql
-- Answer:
-- 创建视图
CREATE VIEW ViewPractice5_1 (product_name, sale_price, regist_date)
AS
SELECT product_name, sale_price, regist_date
FROM product
WHERE sale_price >= 1000 AND regist_date = '2009-09-20';
-- 查看视图
SELECT * FROM ViewPractice5_1;
```



```
3.2
向习题一中创建的视图 ViewPractice5_1 中插入如下数据，会得到什么样的结果呢？

INSERT INTO ViewPractice5_1 VALUES (' 刀子 ', 300, '2009-11-02');
```

```
-- Answer:
“ERROR 1423 (HY000): Field of view 'shop.ViewPractice5_1' underlying table doesn't have a default value”
-- Summary:
对视图的操作就是对底层基础表的操作，底层基本表中有定义才能成功修改，而'刀子'在底层表中无定义，无法修改。
```



```
3.3
请根据如下结果编写 SELECT 语句，其中 sale_price_all 列为全部商品的平均销售单价。

product_id | product_name | product_type | sale_price | sale_price_all
------------+-------------+--------------+------------+---------------------
0001       | T恤衫         | 衣服         | 1000       | 2097.5000000000000000
0002       | 打孔器        | 办公用品      | 500        | 2097.5000000000000000
0003       | 运动T恤       | 衣服          | 4000      | 2097.5000000000000000
0004       | 菜刀          | 厨房用具      | 3000       | 2097.5000000000000000
0005       | 高压锅        | 厨房用具      | 6800       | 2097.5000000000000000
0006       | 叉子          | 厨房用具      | 500        | 2097.5000000000000000
0007       | 擦菜板        | 厨房用具       | 880       | 2097.5000000000000000
0008       | 圆珠笔        | 办公用品       | 100       | 2097.5000000000000000
```

```mysql
-- Answer:
-- 把(标量)子查询的结果，单独放一列
SELECT product_id, product_name, product_type, sale_price,
       (SELECT AVG(sale_price) FROM product) AS sale_price_all
FROM product;
```



```
3.4
请根据习题一中的条件编写一条 SQL 语句，创建一幅包含如下数据的视图（名称为AvgPriceByType）。

product_id | product_name | product_type | sale_price | avg_sale_price
------------+-------------+--------------+------------+---------------------
0001       | T恤衫         | 衣服         | 1000       |2500.0000000000000000
0002       | 打孔器         | 办公用品     | 500        | 300.0000000000000000
0003       | 运动T恤        | 衣服        | 4000        |2500.0000000000000000
0004       | 菜刀          | 厨房用具      | 3000        |2795.0000000000000000
0005       | 高压锅         | 厨房用具     | 6800        |2795.0000000000000000
0006       | 叉子          | 厨房用具      | 500         |2795.0000000000000000
0007       | 擦菜板         | 厨房用具     | 880         |2795.0000000000000000
0008       | 圆珠笔         | 办公用品     | 100         | 300.0000000000000000
提示：其中的关键是 avg_sale_price 列。与习题三不同，这里需要计算出的 是各商品种类的平均销售单价。这与使用关联子查询所得到的结果相同。 也就是说，该列可以使用关联子查询进行创建。问题就是应该在什么地方使用这个关联子查询。
```

```mysql
-- Answer:
-- 创建视图
CREATE VIEW AvgPriceByType (product_id, product_name, product_type, sale_price, sale_price_all)
AS
SELECT product_id, product_name, product_type, sale_price,
       (SELECT AVG(sale_price) 
        FROM product AS p2 
        WHERE p1.product_type = p2.product_type 
        GROUP BY product_type
       ) 
        AS sale_price_all
FROM product AS p1;

-- 查看视图
SELECT * FROM AvgPriceByType;

```



```
3.5
运算或者函数中含有 NULL 时，结果全都会变为NULL ？（判断题）
```

```mysql
-- Answer:
不是所有的运算或者函数都不能处理NULL。
例如，COALESCE函数就是为了将NULL变成非NULL的值，而IS NULL/IS NOT NULL返回一个真值。
```



```
3.6
对本章中使用的 product（商品）表执行如下 2 条 SELECT 语句，能够得到什么样的结果呢？

①
SELECT product_name, purchase_price
  FROM product
 WHERE purchase_price NOT IN (500, 2800, 5000);
②
SELECT product_name, purchase_price
  FROM product
 WHERE purchase_price NOT IN (500, 2800, 5000, NULL);
```

```mysql
-- Answer:
①
+--------------+----------------+
| product_name | purchase_price |
+--------------+----------------+
| 打孔器        |            320 |
| 擦菜板        |            790 |
+--------------+----------------+

②
Empty set (0.01 sec)

-- Summary
使用IN 和 NOT IN 时是无法选取出NULL数据的。
NULL 只能使用 IS NULL 和 IS NOT NULL 来进行判断。
```



```
3.7
按照销售单价（ sale_price）对练习 6.1 中的 product（商品）表中的商品进行如下分类。

低档商品：销售单价在1000日元以下（T恤衫、办公用品、叉子、擦菜板、 圆珠笔）
中档商品：销售单价在1001日元以上3000日元以下（菜刀）
高档商品：销售单价在3001日元以上（运动T恤、高压锅）

请编写出统计上述商品种类中所包含的商品数量的 SELECT 语句，结果如下所示。
执行结果
low_price | mid_price | high_price
----------+-----------+------------
        5 |         1 |         2
```

```mysql
-- Answer:
SELECT SUM(CASE WHEN sale_price <= 1000 THEN 1 ELSE 0 END) AS low_price,
       SUM(CASE WHEN sale_price > 1000 AND sale_price <= 3000 THEN 1 ELSE 0 END) AS mid_price,
       SUM(CASE WHEN sale_price > 3000 THEN 1 ELSE 0 END) AS high_price
FROM product;
```

