# 06 Mysql OLAP

## 6.1 Online Anallytical Processing 窗口函数

```mysql
-- 窗口函数也称为OLAP函数，即OnLine Analytical Processing，即对数据库数据进行实时分析处理
-- SELECT语句都是对整张表进行查询
-- 窗口函数可以让我们有选择的去某一部分数据进行汇总、计算和排序。
-- 窗口函数的通用形式：
<窗口函数> OVER ([PARTITION BY <列名>]
                     ORDER BY <排序用列名>)  
```

### 6.1.1 窗口函数的基本使用

```mysql
-- 窗口函数中关键字PARTITON BY和ORDER BY的作用
-- PARTITON BY
	-- 类似于GROUP BY 子句的分组功能
	-- 但不具备GROUP BY 子句的汇总功能，并不会改变原始表中记录的行数
	-- PARTITION BY能够设定窗口对象范围，用来区分要看哪个窗口

-- ORDER BY
	-- 指定按照哪一列、何种顺序进行排序
	-- 窗口函数中的ORDER BY与SELECT语句末尾的ORDER BY相似
	-- 可以通过关键字ASC/DESC来指定升序/降序，默认按照ASC

SELECT product_name
       ,product_type
       ,sale_price
       ,RANK() OVER (PARTITION BY product_type
                         ORDER BY sale_price) AS ranking
FROM product; 
```

## 6.2 窗口的种类

```
-- 将SUM、MAX、MIN等聚合函数用在窗口函数中
-- RANK、DENSE_RANK、ROW_NUMBER等排序用的专用窗口函数
```

### 6.2.1 专用窗口函数

```mysql
-- RANK函数
	-- 计算排序时，如果存在相同位次的记录，则会跳过之后的位次。
	-- 有 3 条记录排在第 1 位时：1 位、1 位、1 位、4 位……
-- DENSE_RANK函数
	-- 计算排序，即使存在相同位次的记录，也不会跳过之后的位次。
	-- 有 3 条记录排在第 1 位时：1 位、1 位、1 位、2 位……
-- ROW_NUMBER函数
	-- 赋予唯一的连续位次。
	-- 有 3 条记录排在第 1 位时：1 位、2 位、3 位、4 位
	
SELECT  product_name
       ,product_type
       ,sale_price
       ,RANK() OVER (ORDER BY sale_price) AS ranking
       ,DENSE_RANK() OVER (ORDER BY sale_price) AS dense_ranking
       ,ROW_NUMBER() OVER (ORDER BY sale_price) AS row_num
FROM product; 

```

### 6.2.2 聚合函数在窗口函数上的使用

```mysql
-- 使用方法和专用窗口函数一样，只是出来的不是排序而是累计的聚合函数的值
-- 其实是按排序以后，累计到当前行后进行聚合，即合计或取均值
SELECT  product_id,product_name,sale_price,
        SUM(sale_price) OVER (ORDER BY product_id) AS current_sum,
        AVG(sale_price) OVER (ORDER BY product_id) AS current_avg  
FROM product; 


+------------+--------------+------------+-------------+
| product_id | product_name | sale_price | current_sum |
+------------+--------------+------------+-------------+
| 0001       | T恤          |       1000 |        1000 |  <- 1000
| 0002       | 打孔器       |        500 |        1500 |   <- 1000+500
| 0003       | 运动T恤      |       4000 |        5500 |   <- 1000+500+4000
| 0004       | 菜刀         |       3000 |        8500 |   <- 1000+500+4000+3000
| 0005       | 高压锅       |       6800 |       15300 |
| 0006       | 叉子         |        500 |       15800 |
| 0007       | 擦菜板       |        880 |       16680 |
| 0008       | 圆珠笔       |        100 |       16780 |
+------------+--------------+------------+-------------+

+------------+--------------+------------+-------------+
| product_id | product_name | sale_price | current_avg |
+------------+--------------+------------+-------------+
| 0001       | T恤          |       1000 |   1000.0000 | <- (1000)/1
| 0002       | 打孔器       |        500 |    750.0000 |  <- (1000+500)/2
| 0003       | 运动T恤      |       4000 |   1833.3333 |  <- (1000+500+4000)/3
| 0004       | 菜刀         |       3000 |   2125.0000 |  <- (1000+500+4000+3000)/4
| 0005       | 高压锅       |       6800 |   3060.0000 |
| 0006       | 叉子         |        500 |   2633.3333 |
| 0007       | 擦菜板       |        880 |   2382.8571 |
| 0008       | 圆珠笔       |        100 |   2097.5000 |
+------------+--------------+------------+-------------+

```



## 6.3 窗口函数的的应用 - 计算移动平均

```mysql
-- 聚合函数在窗口函数使用时，计算的是累积到当前行的所有的数据的聚合
-- 实际上可以指定更加详细的汇总范围，成为frame（框架）

-- PRECEDING（“之前”）， 将框架指定为 “截止到之前 n 行”，加上自身行
-- FOLLOWING（“之后”）， 将框架指定为 “截止到之后 n 行”，加上自身行
<窗口函数> OVER (ORDER BY <排序用列名>
                 ROWS n PRECEDING )  

-- BETWEEN 1 PRECEDING AND 1 FOLLOWING，将框架指定为 “之前1行” + “之后1行” + “自身”
<窗口函数> OVER (ORDER BY <排序用列名>
                 ROWS BETWEEN n PRECEDING AND n FOLLOWING)


SELECT  product_id,product_name,sale_price,
        AVG(sale_price) OVER (ORDER BY product_id
                               ROWS 2 PRECEDING) AS moving_avg,
        AVG(sale_price) OVER (ORDER BY product_id
                               ROWS BETWEEN 1 PRECEDING 
                                        AND 1 FOLLOWING) AS moving_avg  
FROM product 
```

### 6.3.1 窗口函数适用范围和注意事项

```mysql
-- 窗口函数只能在SELECT子句中使用。
-- 窗口函数OVER中的ORDER BY子句并不会影响最终结果的排序。
-- OVER中的ORDER BY子句只是用来决定窗口函数按何种顺序计算。
```



## 6.4 GROUPING运算符

### 6.4.1 ROLLUP - 计算合计及小计

```mysql
-- 常规的GROUP BY 只能得到每个分类的小计
-- 计算分类的合计，可以用 ROLLUP关键字

-- ROLLUP 对product_type, regist_date两列进行合计汇总。

SELECT  product_type
       ,regist_date
       ,SUM(sale_price) AS sum_price
FROM product
GROUP BY product_type, regist_date WITH ROLLUP  

+--------------+-------------+-----------+
| product_type | regist_date | sum_price |
+--------------+-------------+-----------+
| 办公用品     | 2009-09-11  |       500 |
| 办公用品     | 2009-11-11  |       100 |
| 办公用品     | NULL        |       600 |  <- 小计(办公用品)
| 厨房用具     | 2008-04-28  |       880 |
| 厨房用具     | 2009-01-15  |      6800 |
| 厨房用具     | 2009-09-20  |      3500 |
| 厨房用具     | NULL        |     11180 |  <- 小计(厨房用具)
| 衣服         | NULL        |      4000 |
| 衣服         | 2009-09-20  |      1000 |
| 衣服         | NULL        |      5000 |  <- 小计(衣服)
| NULL         | NULL        |     16780 | <- 合计
+--------------+-------------+-----------+

-- 实际上有三层聚合
-- 模块3是常规的 GROUP BY 的结果，需要注意的是衣服 有个注册日期本身为空的。
-- 模块2和1是 ROLLUP 带来的合计。
-- 模块2是对产品种类的合计。
-- 模块1是对全部数据的总计。

| product_type | regist_date | sum_price |
+--------------+-------------+-----------+
| NULL         | NULL        |     16780 | 模块1
+--------------+-------------+-----------------
| 办公用品     | NULL        |       600 |
| 厨房用具     | NULL        |     11180 | 
| 衣服         | NULL        |      4000 |  模块2
+--------------+-------------+-----------------
| 办公用品     | 2009-09-11  |       500 |
| 办公用品     | 2009-11-11  |       100 |
| 厨房用具     | 2008-04-28  |       880 |
| 厨房用具     | 2009-01-15  |      6800 |
| 厨房用具     | 2009-09-20  |      3500 |
| 衣服         | 2009-09-20  |      1000 |
| 衣服         | NULL        |      5000 |  模块3
+--------------+-------------+-----------------
```



## Practice

```mysql
5.1
-- 请说出针对本章中使用的 product（商品）表执行如下 SELECT 语句所能得到的结果。
SELECT  product_id
       ,product_name
       ,sale_price
       ,MAX(sale_price) OVER (ORDER BY product_id) AS Current_max_price
FROM product
```

```
-- Answer:
按product_id进行排序，然后从上到下，给出到当前列,出现的最高的sale_price
```



```
5.2
继续使用product表，计算出按照登记日期（regist_date）升序进行排列的各日期的销售单价（sale_price）的总额。排序是需要将登记日期为NULL 的“运动 T 恤”记录排在第 1 位（也就是将其看作比其他日期都早）
```

```mysql
-- Answer:
SELECT  regist_date,
        SUM(sale_price) AS Current_sum
FROM product
GROUP BY regist_date WITH ROLLUP;
```



```
5.3
思考题

① 窗口函数不指定PARTITION BY的效果是什么？

② 为什么说窗口函数只能在SELECT子句中使用？实际上，在ORDER BY 子句使用系统并不会报错。
```

```mysql
-- Answer:
① 不指定的话，只按照ORDER BY的列来全局排序。
如果指定了PARTITION BY，则会按照指定的列进行细分排序

② 执行顺序FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
窗口函数是为了动态分析处理，在SELECT子句可以进行筛选，而ORDER BY子句中只是用来排序，无法实现相应功能。
```

