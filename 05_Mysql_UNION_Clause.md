# 05 UNION Clause 集合运算

```
- set 集合在数据库领域表示记录的集合。
- 表、视图和查询的执行结果都是记录的集合。
- 元素为表、视图和查询结果中的每一行。
- 集合运算符 交/并/差
	- UNION 
	- INTERSECT
	- EXCEPT
- 与数学概念set不同，数据库中用bag模型描述更为合适
	- bag允许有元素重复出现
	- bag取并集时的逻辑顺序为：
		- 该元素是否至少在一个bag里出现过。
		- 该元素在两个bag中的最大出现次数
		- 例如 A = {1,1,1,2,3,5,7}, B = {1,1,2,2,4,6,8}
		- A并B的bag模型结果为 {1,1,1,2,2,3,4,5,6,7,8}
	- bag取交集时的逻辑顺序为：
		- 该元素是否同时属于两个bag
		- 该元素在两个bag中的最小出现次数
		- A交B的bag模型结果为 {1,1,2}
	- bag取差集的逻辑顺序为:
		- 该元素是否属于作为被减数的bag
		- 该元素在两个bag中的出现次数
		- A差B的bag模型结果为 {1,3,5,7}
```

![图片来源于网络](https://cdn.jsdelivr.net/gh/WilliamCharlieBlue/img@main/img/05b62bc4e0974ecdf01957f3e84ae613746cbb3e.png)

## 5.1 表的加减法

### 5.1.1 表的加法  UNION

```mysql
-- UNION 取并集，会去除重复记录
-- 对不同表可以进行并集运算
-- 其实同一张表也可以进行求并集，这样的操作和OR就很相似

SELECT product_id, product_name  FROM product
UNION
SELECT product_id, product_name  FROM product2;
```

### 5.1.2 UNION与OR

```mysql
-- 对于同一张表，取并集可以用UNION和OR，由于条件筛选的顺序不一致，结果的顺序有偏差
-- 而不同的表，只能使用UNION

-- 使用OR谓语
SELECT * 
FROM product 
WHERE sale_price / purchase_price< 1.3
OR sale_price / purchase_price IS NULL;

-- 使用UNION
SELECT * 
FROM product 
WHERE sale_price / purchase_price< 1.3
UNION
SELECT * 
FROM product 
WHERE sale_price / purchase_price IS NULL;
```

### 5.1.2 UNION ALL 包含重复行的集合运算

```mysql
-- UNION ALL 在取并集的时候就会保留重复行

SELECT product_id, product_name  FROM product
UNION ALL
SELECT product_id, product_name  FROM product2;
```

### 5.1.3 隐式类型转换

```mysql
-- UNION时，如果类型不一样，会通过隐式类型转换的方式展示在一列里
-- 如第一个SELECT中的字符串类型‘1’，它的结果的每一列都会列出来
SELECT product_id, product_name, '1'
  FROM product
 UNION
SELECT product_id, product_name,sale_price
  FROM product2;
```

```mysql
- SYSDATE()函数可以返回当前日期时间,类型为一个日期时间类型的数据,
- 测试该数据类型和数值,字符串等类型的兼容性。
- 结果可得：时间日期类型和字符串,数值以及缺失值均能兼容

SELECT SYSDATE(), SYSDATE(), SYSDATE()
UNION
SELECT 'chars', 123,  null;

+---------------------+---------------------+---------------------+
| SYSDATE()           | SYSDATE()           | SYSDATE()           |
+---------------------+---------------------+---------------------+
| 2020-12-22 20:36:13 | 2020-12-22 20:36:13 | 2020-12-22 20:36:13 |
| chars               | 123                 | NULL                |
+---------------------+---------------------+---------------------+
```

### 5.1.4  MySQL 8.0 不支持交运算INTERSECT 和差运算EXCEPT

```mysql
-- 运行失败
SELECT product_id, product_name  FROM product
INTERSECT
SELECT product_id, product_name  FROM product2;

-- 运行失败
SELECT product_id, product_name  FROM product
EXCEPT
SELECT product_id, product_name  FROM product2;
```

### 5.1.5 INTERSECT 与 AND 谓词

```mysql
-- 对同一个表来说，INTERSECT可以等价于AND
-- 查找product表中利润率高于50%,并且售价低于1500的商品：
SELECT * 
  FROM product
 WHERE sale_price > 1.5 * purchase_price 
   AND sale_price < 1500
```

### 5.1.6 EXCEPT 与 NOT IN 谓词

```mysql
-- 对同一个表来说，EXCEPT可以等价于NOT IN
-- 查找售价高于2000，且利润不低于30%的商品：
SELECT * 
  FROM product
 WHERE sale_price > 2000 
   AND product_id NOT IN (SELECT product_id 
                            FROM product 
                           WHERE sale_price<1.3*purchase_price)
```

### 5.1.7 对称差与迂回实现多表INTERSECT

```mysql
-- A和B的对称差为：A特有 + B特有
-- 在其他数据库中，(UNION)EXCEPT(INTERSECT) 实现对称差
-- Mysql中使用：(A NOT IN B)UNION(B NOT IN A) 实现对称差
-- Mysql中，用UNION和上述的对称差求多表INTERSECT

-- 使用(NOT IN + 子查询)实现差集
SELECT * FROM product
WHERE product_id NOT IN (SELECT product_id FROM product2)
UNION
SELECT * FROM product2
WHERE product_id NOT IN (SELECT product_id FROM product);

-- 实现多表INTERSECT
SELECT *
-- FROM后面派生出来的表，一定要取别名
FROM (SELECT * FROM product UNION SELECT * FROM product2) AS unionresult
-- 这个WHERE操作的列是product_id，所以NOT IN后面的子查询最终的结果必须是SELECT product_id
-- 往下层子查询也是同样的理由
WHERE product_id NOT IN (SELECT product_id FROM product
                         WHERE product_id NOT IN (SELECT product_id FROM product2)
                         UNION
                         SELECT product_id FROM product2
                         WHERE product_id NOT IN (SELECT product_id FROM product));
                        
```



## 5.2 JOIN 连结

```
- UNION、INTERSECT、EXCEPT只能以行方向为单位进行操作，增减记录的行数
- 而JOIN可以关联多列，特别是在不同表中的列
- JOIN是SQL查询的核心操作
```

### 5.2.1 INNER JOIN 内连结

```mysql
-- 内连结的语法格式：
FROM <tb_1> INNER JOIN <tb_2> ON <condition(s)>
```

```
# 要在product 和 shopproduct之间找一个桥梁，将两张表连起来，这里很明显是product_id
product：
+------------+--------------+--------------+------------+----------------+-------------+
| product_id | product_name | product_type | sale_price | purchase_price | regist_date |
+------------+--------------+--------------+------------+----------------+-------------+
| 0001       | T恤          | 衣服         |       1000 |            500 | 2009-09-20  |
| 0002       | 打孔器       | 办公用品     |        500 |            320 | 2009-09-11  |
| 0003       | 运动T恤      | 衣服         |       4000 |           2800 | NULL        |
| 0004       | 菜刀         | 厨房用具     |       3000 |           2800 | 2009-09-20  |
| 0005       | 高压锅       | 厨房用具     |       6800 |           5000 | 2009-01-15  |
| 0006       | 叉子         | 厨房用具     |        500 |           NULL | 2009-09-20  |
| 0007       | 擦菜板       | 厨房用具     |        880 |            790 | 2008-04-28  |
| 0008       | 圆珠笔       | 办公用品     |        100 |           NULL | 2009-11-11  |
+------------+--------------+--------------+------------+----------------+-------------+

shopproduct：
+---------+-----------+------------+----------+
| shop_id | shop_name | product_id | quantity |
+---------+-----------+------------+----------+
| 000A    | 东京      | 0001       |       30 |
| 000A    | 东京      | 0002       |       50 |
| 000A    | 东京      | 0003       |       15 |
| 000B    | 名古屋    | 0002       |       30 |
| 000B    | 名古屋    | 0003       |      120 |
| 000B    | 名古屋    | 0004       |       20 |
| 000B    | 名古屋    | 0006       |       10 |
| 000B    | 名古屋    | 0007       |       40 |
| 000C    | 大阪      | 0003       |       20 |
| 000C    | 大阪      | 0004       |       50 |
| 000C    | 大阪      | 0006       |       90 |
| 000C    | 大阪      | 0007       |       70 |
| 000D    | 福冈      | 0001       |      100 |
+---------+-----------+------------+----------+
```

#### 5.2.1.1 内连结要点

```mysql
-- 要点一
	-- FROM子句中将两张表连结为一张
	-- FROM shopproduct AS SP INNER JOIN product AS P
-- 要点二
	-- 必须使用ON来指定连接条件，作用与WHERE比较类似
	-- 如果不指定筛选出来的结果是错误的，冗余的
-- 要点三
	-- SELECT 子句中的列最好按照 表名.列名 的格式来使用
	-- 特别是两张表有多个同名列的时候，不指定列名会报错

SELECT SP.shop_id,SP.shop_name,SP.product_id
       ,P.product_name,P.product_type,P.sale_price
       ,SP.quantity
FROM shopproduct AS SP INNER JOIN product AS P
ON SP.product_id = P.product_id;
```

#### 5.2.1.2 内连结 搭配使用WHERE

```mysql
-- 整个内连接派生出一个表，记得一定要取一个别名
SELECT *
FROM (SELECT SP.shop_id,SP.shop_name,SP.product_id
       ,P.product_name,P.product_type,P.sale_price
       ,SP.quantity
      FROM shopproduct AS SP INNER JOIN product AS P
      ON SP.product_id = P.product_id) AS STEP1
WHERE shop_name = '东京' AND product_type = '衣服';

-- 执行顺序FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
-- 但WHERE子句将在FROM子句之后执行，这样就可以利用实现顺序，加速查询
-- 强烈推荐使用
SELECT SP.shop_id,SP.shop_name,SP.product_id
       ,P.product_name,P.product_type,P.sale_price
       ,SP.quantity
FROM shopproduct AS SP INNER JOIN product AS P
ON SP.product_id = P.product_id
WHERE shop_name = '东京' AND product_type = '衣服';

-- 还有一种剑走偏锋，筛选和条件都放在ON语句，不易阅读，不推荐
ON (SP.product_id = P.product_id AND shop_name = '东京' AND product_type = '衣服');

-- 进一步提速，先做筛选再连结，特针复杂的筛选条件时效果显著
-- 虽然语句变复杂了，但是逻辑更清晰，运行更快
-- 实战进阶使用
SELECT SP.shop_id,SP.shop_name,SP.product_id
       ,P.product_name,P.product_type,P.sale_price
       ,SP.quantity
FROM (SELECT * FROM shopproduct WHERE shop_name = '东京')AS SP 
      INNER JOIN 
      (SELECT * FROM product WHERE product_type = '衣服')AS P
ON SP.product_id = P.product_id
WHERE shop_name = '东京' AND product_type = '衣服'
```

#### 5.2.1.3 内连结 搭配使用GROUP BY

```mysql
-- 执行顺序FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
-- 分组列和被聚合的列在同一张表时，可以先使用GROUP BY子句，再连结
-- 注意：分组列和被聚合的列不在同一张表, 且二者都未被用于连结两张表, 只能先连结, 再聚合

-- 每个商店中, 售价最高的商品的售价分别是多少?
SELECT SP.shop_id, SP.shop_name, MAX(P.sale_price) AS max_price
FROM shopproduct AS SP INNER JOIN product AS P
ON SP.product_id = P.product_id
GROUP BY SP.shop_id,SP.shop_name;

-- 每类商品中售价最高的商品都在哪些商店有售?
-- 好像不太对的样子，最后一个 000D 的价格不对
SELECT P.product_type, SP.shop_id, SP.shop_name, MAX(P.sale_price) AS max_price
FROM shopproduct AS SP INNER JOIN product AS P
ON SP.product_id = P.product_id
GROUP BY P.product_type,SP.shop_id,SP.shop_name
ORDER BY P.product_type,SP.shop_id,SP.shop_name;

+--------------+---------+-----------+-----------+
| product_type | shop_id | shop_name | max_price |
+--------------+---------+-----------+-----------+
| 办公用品     | 000A    | 东京      |       500 |
| 办公用品     | 000B    | 名古屋    |       500 |
| 厨房用具     | 000B    | 名古屋    |      3000 |
| 厨房用具     | 000C    | 大阪      |      3000 |
| 衣服         | 000A    | 东京      |      4000 |
| 衣服         | 000B    | 名古屋    |      4000 |
| 衣服         | 000C    | 大阪      |      4000 |
| 衣服         | 000D    | 福冈      |      1000 |
+--------------+---------+-----------+-----------+
```





## Practice

```
4.1
假设连锁店想要增加毛利率超过 50%或者售价低于 800 的货物的存货量, 请使用 UNION 对分别满足上述两个条件的商品的查询结果求并集.

结果应该类似于:
+------------+--------------+--------------+------------+----------------+
| product_id | product_name | product_type | sale_price | purchase_price |
+------------+--------------+--------------+------------+----------------+
| 0001       | T恤          | 衣服         |       1000 |            500 |
| 0002       | 打孔器       | 办公用品     |        500 |            320 |
| 0006       | 叉子         | 厨房用具     |        500 |           NULL |
| 0008       | 圆珠笔       | 办公用品     |        100 |           NULL |
+------------+--------------+--------------+------------+----------------+
```

```mysql
-- Answer:
-- Method1 使用WHERE + OR
SELECT  product_id,product_name,product_type,sale_price,purchase_price
FROM product
WHERE sale_price > 1.5 * purchase_price
OR sale_price < 800;

-- Method2 按两条SELECT语句执行，使用UNION
SELECT  product_id,product_name,product_type,sale_price,purchase_price
FROM product
WHERE sale_price > 1.5 * purchase_price
UNION
SELECT  product_id,product_name,product_type,sale_price,purchase_price
FROM product
WHERE sale_price < 800;
```



```
4.2
找出 product 和 product2 中售价高于 500 的商品的基本信息.
```

```mysql
-- Answer:
SELECT product_id,product_name,product_type,sale_price,purchase_price  
FROM product
WHERE sale_price > 500
UNION
SELECT product_id,product_name,product_type,sale_price,purchase_price 
FROM product2
WHERE sale_price > 500;
```



```
4.3
商店决定对product表中利润不低于50%和售价低于1000的商品提价, 请使用UNION ALL 语句将分别满足上述两个条件的结果取并集. 查询结果类似下表:
+------------+--------------+--------------+------------+----------------+-------------+
| product_id | product_name | product_type | sale_price | purchase_price | regist_date |
+------------+--------------+--------------+------------+----------------+-------------+
| 0002       | 打孔器       | 办公用品     |        500 |            320 | 2009-09-11  |
| 0006       | 叉子         | 厨房用具     |        500 |           NULL | 2009-09-20  |
| 0007       | 擦菜板       | 厨房用具     |        880 |            790 | 2008-04-28  |
| 0008       | 圆珠笔       | 办公用品     |        100 |           NULL | 2009-11-11  |
| 0001       | T恤          | 衣服         |       1000 |            500 | 2009-09-20  |
| 0002       | 打孔器       | 办公用品     |        500 |            320 | 2009-09-11  |
+------------+--------------+--------------+------------+----------------+-------------+
```

```mysql
-- Answer:
SELECT product_id,product_name,product_type,sale_price,purchase_price,regist_date
FROM product
WHERE sale_price < 1000
UNION ALL
SELECT product_id,product_name,product_type,sale_price,purchase_price,regist_date
FROM product
WHERE sale_price > 1.5 * purchase_price;
```



```
4.4
借助对称差的实现方式, 求product和product2的交.
```

```mysql
-- Answer:
SELECT *
-- FROM后面派生出来的表，一定要取别名
FROM (SELECT * FROM product UNION SELECT * FROM product2) AS unionresult
-- 这个WHERE操作的列是product_id，所以NOT IN后面的子查询最终的结果必须是SELECT product_id
-- 往下层子查询也是同样的理由
WHERE product_id NOT IN (SELECT product_id FROM product
                         WHERE product_id NOT IN (SELECT product_id FROM product2)
                         UNION
                         SELECT product_id FROM product2
                         WHERE product_id NOT IN (SELECT product_id FROM product));
```



```
4.5
找出每个商店里的衣服类商品的名称及价格等信息
+---------+-----------+------------+--------------+--------------+------------+----------+
| shop_id | shop_name | product_id | product_name | product_type | sale_price | quantity |
+---------+-----------+------------+--------------+--------------+------------+----------+
| 000A    | 东京      | 0001       | T恤          | 衣服         |       1000 |       30 |
| 000A    | 东京      | 0003       | 运动T恤      | 衣服         |       4000 |       15 |
| 000B    | 名古屋    | 0003       | 运动T恤      | 衣服         |       4000 |      120 |
| 000C    | 大阪      | 0003       | 运动T恤      | 衣服         |       4000 |       20 |
| 000D    | 福冈      | 0001       | T恤          | 衣服         |       1000 |      100 |
+---------+-----------+------------+--------------+--------------+------------+----------+
```

```mysql
-- Answer:
-- 先连结，再筛选
SELECT SP.shop_id,SP.shop_name,SP.product_id
       ,P.product_name,P.product_type,P.sale_price
       ,SP.quantity
FROM shopproduct AS SP INNER JOIN product AS P
ON SP.product_id = P.product_id
WHERE product_type = '衣服';

-- 先筛选，再连结
SELECT SP.shop_id,SP.shop_name,SP.product_id
       ,P.product_name,P.product_type,P.sale_price
       ,SP.quantity
-- 为了更快一些，可以 SELECT product_id, product_name, product_type, purchase_price
FROM shopproduct AS SP INNER JOIN (SELECT * FROM product WHERE product_type = '衣服') AS P
ON SP.product_id = P.product_id;

```



```
4.6 
分别使用连结两个子查询和不使用子查询的方式, 找出东京商店里, 售价低于 2000 的商品信息,希望得到如下结果
+---------+-----------+------------+----------+------------+--------------+--------------+------------+----------------+-------------+
| shop_id | shop_name | product_id | quantity | product_id | product_name | product_type | sale_price | purchase_price | regist_date |
+---------+-----------+------------+----------+------------+--------------+--------------+------------+----------------+-------------+
| 000A    | 东京      | 0001       |       30 | 0001       | T恤          | 衣服         |       1000 |            500 | 2009-09-20  |
| 000A    | 东京      | 0002       |       50 | 0002       | 打孔器       | 办公用品     |        500 |            320 | 2009-09-11  |
+---------+-----------+------------+----------+------------+--------------+--------------+------------+----------------+-------------+
```

```mysql
-- Answer:
-- 不进行子查询
SELECT SP.*, P.*
FROM shopproduct AS SP INNER JOIN product AS P 
ON SP.product_id = P.product_id
WHERE shop_id = '000A' AND sale_price < 2000;
-- 两个子查询
SELECT SP.*, P.*
FROM (SELECT * FROM shopproduct WHERE shop_id = '000A') AS SP 
     INNER JOIN  
     (SELECT * FROM product WHERE sale_price < 2000) AS P
ON SP.product_id = P.product_id;

```



