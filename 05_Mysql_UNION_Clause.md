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
-- 好像不太对的样子，直接内连结，不好做筛选和排序，逻辑很混乱
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

-- 每类商品中售价最高的商品都在哪些商店有售?
-- 这个貌似还是不对，两次内连结还是出了问题，缺少了筛选的步骤
SELECT P.product_type, SP.shop_id, SP.shop_name, P.max_price
FROM shopproduct AS SP 
     INNER JOIN 
     ( SELECT P1.product_type,P1.product_id, P1.product_name,P2.max_price
         FROM product AS P1
            INNER JOIN 
            (SELECT product_type,MAX(sale_price) AS max_price 
                   FROM product GROUP BY product_type) AS P2  
                   ON P1.product_type = P2.product_type 
             ) AS P
ON SP.product_id = P.product_id
GROUP BY P.product_type,SP.shop_id,SP.shop_name
ORDER BY P.product_type,SP.shop_id,SP.shop_name;
+--------------+---------+-----------+-----------+
| product_type | shop_id | shop_name | max_price |
+--------------+---------+-----------+-----------+
| 办公用品     | 000A    | 东京      |       500 |
| 办公用品     | 000B    | 名古屋    |       500 |
| 厨房用具     | 000B    | 名古屋    |      6800 |
| 厨房用具     | 000C    | 大阪      |      6800 |
| 衣服         | 000A    | 东京      |      4000 |
| 衣服         | 000B    | 名古屋    |      4000 |
| 衣服         | 000C    | 大阪      |      4000 |
| 衣服         | 000D    | 福冈      |      4000 |
+--------------+---------+-----------+-----------+

-- 每类商品中售价最高的商品都在哪些商店有售?
-- 终于成功了
SELECT P.product_type, P.product_name, SP.shop_id, SP.shop_name, P.sale_price
FROM shopproduct AS SP 
     INNER JOIN
     -- 使用关联子查询得到每一种商品种类的最高价格,派生出一个新表P 与SP进行内连结
     (SELECT product_type, product_id, product_name, sale_price 
     FROM product AS P1 
     WHERE sale_price = (SELECT MAX(sale_price)                        
                    FROM product AS P2                        
                    WHERE P1.product_type = P2.product_type                       
                    GROUP BY product_type)) AS P
ON SP.product_id = P.product_id
ORDER BY P.product_type;
-- 厨房用品中最高价的高压锅，这几家店都没有
+--------------+--------------+---------+-----------+------------+
| product_type | product_name | shop_id | shop_name | sale_price |
+--------------+--------------+---------+-----------+------------+
| 办公用品     | 打孔器       | 000A    | 东京      |        500 |
| 办公用品     | 打孔器       | 000B    | 名古屋    |        500 |
| 衣服         | 运动T恤      | 000A    | 东京      |       4000 |
| 衣服         | 运动T恤      | 000B    | 名古屋    |       4000 |
| 衣服         | 运动T恤      | 000C    | 大阪      |       4000 |
+--------------+--------------+---------+-----------+------------+
```

#### 5.2.1.4 内连结 与 关联子查询

```mysql
-- 关联子查询中的问题：找出每个商品种类当中售价高于该类商品的平均售价的商品
SELECT product_type, product_name, sale_price
FROM product AS P1
WHERE sale_price > (SELECT AVG(sale_price)
                       FROM product AS P2
                      WHERE P1.product_type = P2.product_type
                      GROUP BY product_type);
-- 使用内连结来实现
SELECT  P1.product_id,P1.product_name,P1.product_type,P1.sale_price
       ,P2.avg_price
FROM product AS P1 
     INNER JOIN 
    (SELECT product_type,AVG(sale_price) AS avg_price 
      FROM product 
     GROUP BY product_type) AS P2 
ON P1.product_type = P2.product_type
WHERE P1.sale_price > P2.avg_price;

-- 去掉内连结的子查询，虽然看起来层次更少，代码行数更少。但更难编写
-- 往往需要耗费很长的时间，写一个虽然可能效率高，但晦涩难懂的代码，不太推荐
SELECT  P1.product_id,P1.product_name,P1.product_type,P1.sale_price
       ,AVG(P2.sale_price) AS avg_price
FROM product AS P1 INNER JOIN product AS P2 
ON P1.product_type=P2.product_type
WHERE P1.sale_price > P2.sale_price
GROUP BY P1.product_id,P1.product_name,P1.product_type,P1.sale_price,P2.product_type;
```

#### 5.2.1.5 NATURAL JOIN 自然连结

```mysql
-- 自然连结是内连结的一种特例
-- 按照两个表中都包含的列名来进行等值内连结
-- 比如shopproduct 和 product公共列是product_id，公共列放在第一列
-- 可以有多个公共列，非公共列的依次罗列出来
SELECT *  FROM shopproduct NATURAL JOIN product

-- 使用基础的内连结实现
SELECT P.product_id, SP.shop_id, SP.shop_name, SP.quantity,
       P.product_name, P.product_type, P.sale_price,
       P.purchase_price, P.regist_date
FROM shopproduct AS SP INNER JOIN product AS P
ON SP.product_id = P.product_id;

-- 自然连结可以求出两张表或子查询的公共部分，解决INTERSECT问题
-- 求表 product 和表 product2 中的公共部分？
SELECT * FROM product NATURAL JOIN product2
-- 但是连结只会返回连结条件为真的行，如果是NULL的话，不能用"="来判断
-- 因此需要避开可能包含NULL的列
SELECT * 
FROM (SELECT product_id, product_name, product_type, sale_price FROM product ) AS A 
      NATURAL JOIN 
     (SELECT product_id, product_name, product_type, sale_price FROM product2) AS B;
```

#### 5.2.1.6 使用内连结求交集

```mysql
-- 如上一节内容，可以通过NATURAL JOIN求INTERSECT
-- 关键在于公共列的等值判断
-- 在INNER JOIN的ON子句中加入足够多的条件，也就能实现INTERSECT
-- 但NULL的问题依旧存在，NULL不能用"="来判断，在ON语句中会判断为假，从而漏掉
SELECT P1.*
FROM product AS P1 INNER JOIN product2 AS P2
ON (P1.product_id  = P2.product_id
   AND P1.product_name = P2.product_name
   AND P1.product_type = P2.product_type
   AND P1.sale_price   = P2.sale_price
   AND P1.regist_date  = P2.regist_date);

-- 通过减少冗余，但包含NULL的条件，即可
SELECT P1.*
FROM product AS P1 INNER JOIN product2 AS P2
ON P1.product_id = P2.product_id;
```



### 5.2.3 OUTER JOIN 外连结

```mysql
-- 内连结会丢弃两张表中不满足ON条件的行
-- 外连结有选择地保留无法匹配到的行
	-- 左连结 保留左表中无法匹配地行，右表的行为缺失填充
	-- 右连结 保留右表中无法匹配地行，左表的行为缺失填充
	-- 全外连结 保留左右两表，对应的另一张表的行为缺失填充
-- 左连结     
FROM <tb_1> LEFT  OUTER JOIN <tb_2> ON <condition(s)>
-- 右连结     
FROM <tb_1> RIGHT OUTER JOIN <tb_2> ON <condition(s)>
-- 全外连结
FROM <tb_1> FULL  OUTER JOIN <tb_2> ON <condition(s)>
```

#### 5.2.3.1 左连结与右链接

```
-- 连结时可以交换左表和右表的位置, 左连结和右连结并没有本质区别
-- 可以在表调换位置，同时左/右连结交换后，结果是一致的
```

#### 5.2.3.2 使用左连接获取信息

```mysql
-- product 表中有两种商品并未在内连结的结果里，说明商品未出售，有可能处于缺货状态
-- 例如高压锅和圆珠笔
SELECT SP.shop_id,SP.shop_name,
       P.product_id, P.product_name,P.sale_price
FROM product AS P
LEFT OUTER JOIN shopproduct AS SP
ON SP.product_id = P.product_id;
+---------+-----------+------------+--------------+------------+
| shop_id | shop_name | product_id | product_name | sale_price |
+---------+-----------+------------+--------------+------------+
| 000D    | 福冈      | 0001       | T恤          |       1000 |
| 000A    | 东京      | 0001       | T恤          |       1000 |
| 000B    | 名古屋    | 0002       | 打孔器       |        500 |
| 000A    | 东京      | 0002       | 打孔器       |        500 |
| 000C    | 大阪      | 0003       | 运动T恤      |       4000 |
| 000B    | 名古屋    | 0003       | 运动T恤      |       4000 |
| 000A    | 东京      | 0003       | 运动T恤      |       4000 |
| 000C    | 大阪      | 0004       | 菜刀         |       3000 |
| 000B    | 名古屋    | 0004       | 菜刀         |       3000 |
| NULL    | NULL      | 0005       | 高压锅       |       6800 |
| 000C    | 大阪      | 0006       | 叉子         |        500 |
| 000B    | 名古屋    | 0006       | 叉子         |        500 |
| 000C    | 大阪      | 0007       | 擦菜板       |        880 |
| 000B    | 名古屋    | 0007       | 擦菜板       |        880 |
| NULL    | NULL      | 0008       | 圆珠笔       |        100 |
+---------+-----------+------------+--------------+------------+
```

#### 5.2.3.3 外连结细节

```
-- 要点一：选出单张表中全部的信息
	-- 不会内连结一样漏掉一些信息
	-- 能生成生成固定行数的单据
	
-- 要点二：使用 LEFT、RIGHT 来指定主表
	-- 指定的主表，最终结果将包含主表内的所有数据
	-- LEFT和RIGHT只是指定的主表的位置不同，没有实质性的差别
	-- 交换两个表的顺序, 同时将LEFT/RIGHT互换，结果完全相同
```

#### 5.2.3.4 左连结 搭配 WHERE

```mysql
-- WHERE进行筛选时，所使用的">=<"都对NULL无效
-- 如果先进行外连结，再使用WHERE，会丢失包含NULL的条目
-- 执行顺序FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
-- 想办法将WHERE在FROM OUTER JOIN之前实现，即使用子查询

-- 先OUTER JION再WHERE，会忽略NULL
SELECT P.product_id, P.product_name, P.sale_price,
       SP.shop_id, SP.shop_name, SP.quantity
FROM product AS P LEFT OUTER JOIN shopproduct AS SP ON SP.product_id = P.product_id
WHERE quantity< 50

-- 让WHERE先执行，再进行外连结，则可保留NULL信息
SELECT P.product_id, P.product_name, P.sale_price,
       SP.shop_id, SP.shop_name, SP.quantity
FROM product AS P 
	 LEFT OUTER JOIN 
	 (SELECT * FROM shopproduct WHERE quantity< 50) AS SP 
ON SP.product_id = P.product_id;
```

#### 5.2.3.4  MySQL 中不支持全外连结

```
-- 太遗憾了，又不支持
-- 可以通过左连结和右连结的结果进行 UNION 来实现全外连结

```



### 5.2.4 SELF JOIN 自连结

```
-- 自连结自成一派的连结方法，而不是和第三种连结方案
-- 自连结可以是内连结也可以是外连结
-- 见5.2.6.1
```



### 5.2.5 多表连结

```
-- 除了2张表的连结外，还可能出现3张表连结的情况
-- 连结表的数量其实没有限制
-- 现在有3张表，product, shopproduct, inventoryproduct
```

#### 5.2.5.1 多表进行内连结

```mysql
-- 直接在INNER JOIN后面再加INNER JOIN
-- 对于ON的条件，不需要三连"=", 两个“=”已满足了三者均相等
-- 对于4张、5张, INNER JOIN也是同样的添加方式
SELECT SP.shop_id,SP.shop_name,SP.product_id,
       P.product_name,P.sale_price,IP.inventory_quantity
FROM product AS P
     INNER JOIN shopproduct AS SP
     ON P.product_id = SP.product_id
     INNER JOIN Inventoryproduct AS IP
     ON P.product_id = IP.product_id
WHERE IP.inventory_id = 'P001';

```

#### 5.2.5.2 多表进行外连结

```mysql
-- 直接在OUTER JOIN后面再加OUTER JOIN
-- 一般情况下，如果使用LEFT，就一直使用LEFT，可以保持主表的完整性
SELECT SP.shop_id,SP.shop_name,SP.product_id,
       P.product_name,P.sale_price,IP.inventory_quantity
FROM product AS P
     LEFT OUTER JOIN shopproduct AS SP
     ON P.product_id = SP.product_id
     LEFT OUTER JOIN Inventoryproduct AS IP
     ON P.product_id = IP.product_id
WHERE IP.inventory_id = 'P001';

```



### 5.2.6 ON 子句进阶–非等值连结

```mysql
-- ON子句中，除了使用相等判断的等值连结, 也可以使用比较运算符来进行连接
-- 比较运算符(<,<=,>,>=, BETWEEN)和谓词运算(LIKE, IN, NOT 等等)在内的所有的逻辑运算
```

#### 5.2.6.1 SELF JOIN 非等值自左连结

```mysql
-- 使用非等值自左连结实现排名。
-- 希望对 product 表中的商品按照售价赋予排名. 一个从集合论出发,使用自左连结的思路是, 对每一种商品,找出售价不低于它的所有商品, 然后对售价不低于它的商品使用 COUNT函数计数. 例如, 对于价格最高的商品

-- 使用自左连结对每种商品找出价格不低于它的商品
SELECT product_id, product_name, sale_price, COUNT(P2_id) AS rank
FROM (SELECT P1.product_id, P1.product_name, P1.sale_price, P2.product_id AS P2_id,P2.product_name AS P2_name,P2.sale_price AS P2_price
      FROM product AS P1 LEFT OUTER JOIN product AS P2 
      ON P1.sale_price <= P2.sale_price) AS P
GROUP BY product_id, product_name, sale_price
ORDER BY rank; 

-- 请按照商品的售价从低到高,对售价进行累计求和[注:这个案例缺少实际意义]
-- 对每种商品使用自左连结, 找出比该商品售价价格更低或相等的商品
-- 有两种商品的售价相同, 在使用 >= 进行连结时, 导致了累计求和错误
-- 建立自左连结的本意, 是要找出满足:
	-- 1.比该商品售价更低的
	-- 2.该种商品自身
	-- 3.如果 A 和 B 两种商品售价相等,则建立连结时, 如果 P1.A 和 P2.A,P2.B 建立了连接, 则 P1.B 不再和 P2.A 建立连结
-- 优化：因此根据上述约束条件, 利用 ID 的有序性,
SELECT	product_id, product_name, sale_price, SUM(P2_price) AS cum_price 
FROM (SELECT  P1.product_id, P1.product_name, P1.sale_price
                ,P2.product_id AS P2_id
                ,P2.product_name AS P2_name
                ,P2.sale_price AS P2_price 
      FROM product AS P1 
      LEFT OUTER JOIN product AS P2 
      ON ((P1.sale_price > P2.sale_price) OR (P1.sale_price = P2.sale_price AND P1.product_id<=P2.product_id))
	  ORDER BY P1.sale_price,P1.product_id) AS X
GROUP BY product_id, product_name, sale_price
ORDER BY sale_price,cum_price;
```



### 5.2.7 CROSS JOIN 交叉连结(笛卡尔积)

```mysql
-- 上述无论是外连结内连结, 一个共同的必备条件就是连结条件–ON 子句, 用来指定连结的条件
-- 笛卡尔积, 就是使用集合 A 中的每一个元素与集合 B 中的每一个元素组成一个有序的组合
-- 数据库表(或者子查询)的并,交和差都是在纵向上对表进行扩张或筛选限制等运算的, 这要求表的列数及对应位置的列的数据类型"相容", 因此这些运算并不会增加新的列
-- 交叉连接(笛卡尔积)则是在横向上对表进行扩张, 即增加新的列, 与连结的功能是一致的
	-- 因为没有了ON子句的限制, 会对左表和右表的每一行进行组合
	-- 经常会导致很多无意义的行出现在检索结果中
	-- 结果没有实用价值，其结果行数太多，需要花费大量的运算时间和高性能设备的支持

-- 交叉连结的语法有如下两种形式:
-- 结果为13 × 8 = 104 条记录
-- 1.使用关键字 CROSS JOIN 显式地进行交叉连结
SELECT SP.shop_id
       ,SP.shop_name
       ,SP.product_id
       ,P.product_name
       ,P.sale_price
  FROM shopproduct AS SP
 CROSS JOIN product AS P;
-- 2.使用逗号分隔两个表,并省略 JOIN 子句
SELECT SP.shop_id
       ,SP.shop_name
       ,SP.product_id
       ,P.product_name
       ,P.sale_price
  FROM shopproduct AS SP , product AS P;
```

#### 5.2.7.1连结与笛卡儿积的关系

```mysql
-- 笛卡儿积可以视作一种特殊的连结
-- 笛卡儿积的语法也可以写作(CROSS JOIN), 这种连结的ON子句是一个恒为真的谓词。
-- 对笛卡儿积进行适当的限制之后, 也就得到了内连结和外连结

-- 不做限制的笛卡尔积
SELECT SP.*, P.*
  FROM shopproduct AS SP 
 CROSS JOIN product AS P;

-- 对product_id进行限制后，得到内连结的结果
SELECT SP.*, P.*
  FROM shopproduct AS SP 
 CROSS JOIN product AS P
 WHERE SP.product_id = P.product_id;

-- 内连结的过时语法 FROM里直接用","分隔，ON的功能全部在WHERE中实现
-- 虽然结果与标准语法相同，所有的DBMS都能执行，但可读性差，已经不推荐了
SELECT SP.shop_id,SP.shop_name,SP.product_id,P.product_name,P.sale_price
  FROM shopproduct SP, product P
 WHERE SP.product_id = P.product_id
   AND SP.shop_id = '000A';
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



```
4.7 
分别使用内连结和关联子查询每一类商品中售价最高的商品.
```

```mysql
-- Answer:
-- 使用关联子查询
SELECT product_type, product_id, product_name, sale_price 
FROM product AS P1 
WHERE sale_price = (SELECT MAX(sale_price)                        
                    FROM product AS P2                        
                    WHERE P1.product_type = P2.product_type                       
                    GROUP BY product_type);
-- 使用内连结
SELECT P1.product_type, P1.product_id, P1.product_name, P2.max_price
FROM product AS P1 
     INNER JOIN
     (SELECT product_type, MAX(sale_price) AS max_price
      FROM product 
      GROUP BY product_type) AS P2
ON P1.product_type = P2.product_type
WHERE P1.sale_price = P2.max_price;
```



```mysql
4.8 
SELECT	product_id, product_name, sale_price, SUM(P2_price) AS cum_price 
FROM (SELECT  P1.product_id, P1.product_name, P1.sale_price
                ,P2.product_id AS P2_id
                ,P2.product_name AS P2_name
                ,P2.sale_price AS P2_price 
      FROM product AS P1 
      LEFT OUTER JOIN product AS P2 
      ON ((P1.sale_price > P2.sale_price) OR (P1.sale_price = P2.sale_price AND P1.product_id<=P2.product_id))
	  ORDER BY P1.sale_price,P1.product_id) AS X
GROUP BY product_id, product_name, sale_price
ORDER BY sale_price,cum_price;
试将上述查询改用关联子查询实现.
+------------+--------------+------------+-----------+
| product_id | product_name | sale_price | cum_price |
+------------+--------------+------------+-----------+
| 0008       | 圆珠笔       |        100 |       100 |
| 0006       | 叉子         |        500 |       600 |
| 0002       | 打孔器       |        500 |      1100 |
| 0007       | 擦菜板       |        880 |      1980 |
| 0001       | T恤          |       1000 |      2980 |
| 0004       | 菜刀         |       3000 |      5980 |
| 0003       | 运动T恤      |       4000 |      9980 |
| 0005       | 高压锅       |       6800 |     16780 |
+------------+--------------+------------+-----------+
```

```mysql
-- Answer:
SELECT	product_id, product_name, sale_price, 
		(select SUM(sale_price) 
         from product AS P2
         where P1.sale_price > P2.sale_price
         	OR ((P1.sale_price = P2.sale_price) AND (P1.product_id<=P2.product_id))
        ) AS cum_price 
FROM product AS P1
ORDER BY sale_price;
```

