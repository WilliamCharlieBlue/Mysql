# 03_Mysql_Query_Sort

## 3.1 基本查询

### 3.1.1 SELECT, FROM, WHERE

FROM表明来源，WHERE筛选条件， SELECT输出结果

```mysql
SELECT <列名>，
  FROM <表名>；
 WHERE <条件表达式>；

/* 三段式查询范例 */
SELECT product_type, product_name
FROM product
WHERE product_type = '衣服';
```

### 3.1.2 查询语句规范
- 星号(*)可匹配所有(列)。
- SQL中以";"作为句末，可随意使用换行符(但不可插入空行)
- AS关键词设定中文别名时需要使用双引号("")，而平常的中文字符可以使用单引号('')
- SELECT DISTINCT 可以删除重复行（针对的是所有）
- SQL注释，单行可使用“-- ”，多行可使用 “ /**/”

```mysql
-- 想要查询出全部列时，可以使用代表所有列的星号（*）。
SELECT *
  FROM product；
/* SQL语句可以使用AS关键字为列设定别名
  （用中文时需要双引号（“”））。*/
SELECT product_id     As id,
       product_name   As name,
       purchase_price AS "进货单价"
  FROM product;
-- 使用DISTINCT删除product_type列中重复的数据
SELECT DISTINCT product_type
  FROM product;
```



## 3.2 Mysql运算符

### 3.2.1 算术运算符

```
+ - * / （加减乘除）
```

### 3.2.2 比较运算符

```
= > >= <= < <>（等于，大于，大于等于，小于等于，小于，不等于）
```

### 3.2.3 逻辑运算符

```
NOT 非
AND 且
OR  或

无NULL真值表：
NOT 真假互换
AND	若同则同，有假必假
OR	若同则同，有真必真

有NULL真值表（三值逻辑下的真值表）：
AND	若同则同，有假必假，不确定强于真
OR 	若同则同，有真必真，不确定强于假
```

### 3.2.4 常用法则

```
- ()括号的优先级最高
	- AND 和 OR 连用时记得加（）确保优先级顺序准确
- 字符串类型的数据原则上按照字典顺序进行排序，不能与数字的大小顺序混淆。
- IS NULL/IS NO NULL为固定搭配，而不是使用=/<>符号
```

```mysql
-- 使用括号确保优先级
SELECT product_name, product_type, regist_date
  FROM product
 WHERE product_type = '办公用品'
   AND ( regist_date = '2009-09-11'
        OR regist_date = '2009-09-20');
-- NULL 和 NOT NULL的使用
SELECT product_name, purchase_price
  FROM product
 WHERE purchase_price IS NULL OR product_name IS NOT NULL;
```



## 3.3 聚合查询

### 3.3.1 聚合函数

```
COUNT：计算表中的记录数（行数）
SUM：计算表中数值列中数据的合计值
AVG：计算表中数值列中数据的平均值
MAX：求出表中任意列中数据的最大值
MIN：求出表中任意列中数据的最小值

进阶：结合DISTINCT查看去重前后的差异
```

```mysql
-- 计算全部数据的行数（包含NULL）
SELECT COUNT(*)
  FROM product;
-- 计算NULL以外数据的行数
SELECT COUNT(purchase_price)
  FROM product;
-- 计算销售单价和进货单价的合计值
SELECT SUM(sale_price), SUM(purchase_price) 
  FROM product;
-- 计算销售单价和进货单价的平均值
SELECT AVG(sale_price), AVG(purchase_price)
  FROM product;
-- MAX和MIN也可用于非数值型数据
SELECT MAX(regist_date), MIN(regist_date)
  FROM product;
  
-- 计算去除重复数据后的数据行数
SELECT COUNT(product_type), COUNT(DISTINCT product_type)
 FROM product;
-- 是否使用DISTINCT时的动作差异（SUM函数）
SELECT SUM(sale_price), SUM(DISTINCT sale_price)
 FROM product;
```

### 3.3.2 聚合函数常用法则

```
- COUNT(*)会包含NULL，而COUNT(<列名>)不统计NULL
- 除COUNT外的聚合函数都排除NULL
- SUM/AVG只适用于数值，而MAX/MIN几乎适用于所有类型
- 与DISTINCT共同使用可删除重复值，或者统计value的种类
```



## 3.4 分组查询

### 3.4.1 GROUP BY语句

```
- GROUP BY按某一列来所有种类对整个表进行分组切分统计。
- NULL 作为特殊的一组数据进行处理（即单独分作一组）

SELECT <列名1>,<列名2>, <列名3>, ……
  FROM <表名>
 GROUP BY <列名1>, <列名2>, <列名3>, ……;
```

```mysql
-- 按product_type值的种类来分组，分别计算每组的个数(行数)
SELECT product_type, COUNT(*)
  FROM product
 GROUP BY product_type;
-- 此时会报错 
SELECT product_type, COUNT(*)
  FROM product

-- NULL单独作为一组处理
SELECT purchase_price, COUNT(*)
  FROM product
 GROUP BY purchase_price;
```

### 3.4.2 GROUP BY语句常见问题

```
- SELECT中单独使用列名，只能使用GROUP BY指定的列名(聚合键)
- GROUP BY中不能使用别名，而SELECT子句中可以通过AS来指定别名
- 若WHERE还未确定结果集，使用聚合函数会产生冲突，正确姿势是在SELECT、HAVING和ORDER BY中使用聚合函数指定条件
```



## 3.5 聚合结果指定条件

### 3.5.1 HAVING的使用

```
- GROUP BY分组后，如何能从这些分组中抽出几组呢
- WHERE做不到，因为它只能指定行的条件，不能指定组的条件
- HAVING则是个好选择，语法与WHERE类似，在GROUP BY后使用
- HAVING中单独使用列名，也一定要是GROUP BY指定的
```

### 3.5.2 HAVING的特点

```
- HAVING子句用于对分组进行过滤，可以使用数字、聚合函数和GROUP BY中指定的列名（聚合键）
```

```mysql
-- 数字
SELECT product_type, COUNT(*)
  FROM product
 GROUP BY product_type
HAVING COUNT(*) = 2;
-- 使用指定列名(聚合键)
SELECT product_type, COUNT(*)
  FROM product
 GROUP BY product_type
-- 错误形式（因为product_name不包含在GROUP BY聚合键中）
-- HAVING product_name = '圆珠笔';
HAVING product_type = '衣服' OR product_type = '办公用品';
```



## 3.6 对查询结果进行排序

### 3.6.1 ORDER BY语句

```
- SQL中的执行结果是随机排列的，排序可使用ORDER BY
- 默认为升序排列ASC，降序排列为DESC
- 多个排序键时，顺序在前的排序键先排序，依次往后
- 当用于排序的列名中含有NULL时，NULL会在开头或末尾进行汇总。

SELECT <列名1>, <列名2>, <列名3>, ……
  FROM <表名>
 ORDER BY <排序基准列1>, <排序基准列2>, ……
```

```mysql
-- 降序排列
SELECT product_id, product_name, sale_price, purchase_price
  FROM product
 ORDER BY sale_price DESC;
-- 多个排序键
SELECT product_id, product_name, sale_price, purchase_price
  FROM product
 ORDER BY sale_price, product_id;
-- 当用于排序的列名中含有NULL时，NULL会在开头或末尾进行汇总。
SELECT product_id, product_name, sale_price, purchase_price
  FROM product
 ORDER BY purchase_price;
```

### 3.6.2 ORDER BY细节

```
- 书写顺序
1.SELECT → 2.FROM → 3.WHERE → 4.GROUP BY → 5.HAVING  → 6.ORDER BY；
- 执行顺序
	- 这就是为什么ORDER BY能使用SELECT设置的别名，而GROUP BY不能
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
- 对于含有NULL的两种情况
	- 若其他要实现DESC，要让NULL在最上面，使用加负号的形式ORDER BY -regist_date ASC;
	- 若其他要实现ASC ，要让NULL在最下面，使用加负号的形式ORDER BY -regist_date DESC;
		- 其他实现ASC ，要让NULL在最下面，ORDER BY  ISNULL(regist_date) ASC, regist_date ASC;
- ISNULL(exper)用法，exper是否为空，是则返回1，否则返回0。如果是升序排列，NULL都会在最下方。
```




## Practice

```
2.1
编写一条SQL语句，从product（商品）表中选取出“登记日期（regist在2009年4月28日之后”的商品，查询结果要包含product name和regist_date两列。
```

```mysql
-- Answer:
SELECT product_name, regist_date
FROM product
WHERE regist_date > '2009-04-28';
```



```
2.2
请说出对product 表执行如下3条SELECT语句时的返回结果。
①
SELECT *
  FROM product
 WHERE purchase_price = NULL;

②
SELECT *
  FROM product
 WHERE purchase_price <> NULL;
 
③
SELECT *
  FROM product
 WHERE product_name > NULL;
```

```mysql
-- Answer:
① Empty set
② Empty set
③ Empty set
Summary: NULL不能使用运算符比较，需要用 IS NULL 或者 IS NOT NULL来表达。
```



```sql
2.3
代码清单2-22（2-2节）中的SELECT语句能够从product表中取出“销售单价（saleprice）比进货单价（purchase price）高出500日元以上”的商品。请写出两条可以得到相同结果的SELECT语句。执行结果如下所示。

product_name | sale_price | purchase_price 
-------------+------------+------------
T恤衫         |   1000    | 500
运动T恤       |    4000    | 2800
高压锅        |    6800    | 5000
```

```mysql
-- Answer:
-- Method1
SELECT product_name,sale_price,purchase_price 
FROM product  
WHERE sale_price >= purchase_price + 500;

-- Method2
SELECT product_name,sale_price,purchase_price 
FROM product  
WHERE sale_price - 500 >= purchase_price;
```



```
2.4
请写出一条SELECT语句，从product表中选取出满足“销售单价打九折之后利润高于100日元的办公用品和厨房用具”条件的记录。查询结果要包括product_name列、product_type列以及销售单价打九折之后的利润（别名设定为profit）。

提示：销售单价打九折，可以通过saleprice列的值乘以0.9获得，利润可以通过该值减去purchase_price列的值获得。
```

```mysql
-- Answer:
SELECT product_name, product_type, sale_price*0.9 - purchase_price AS "profit"
FROM product
WHERE (product_type = '办公用品' OR product_type = '厨房用具' )
  AND sale_price*0.9 - purchase_price > 100;
```



```mysql
2.5
请指出下述SELECT语句中所有的语法错误。

SELECT product_id, SUM（product_name）
-- 本SELECT语句中存在错误。
  FROM product 
 GROUP BY product_type 
 WHERE regist_date > '2009-09-01';
```

```mysql
-- Answer:
-- 1、书写顺序错误，正确的书写顺序为 1.SELECT → 2.FROM → 3.WHERE → 4.GROUP BY；
-- 2、使用GROUP BY时，SELECT中的单独列名只能是GROUP BY的指定列名。可以改product_id为product_type；
-- 3、SUM/AVG只使用于数值类型，而product_name是字符类型。可以改SUM为COUNT。

SELECT product_type, COUNT(product_name) 
FROM product  
WHERE regist_date > '2009-09-01' 
GROUP BY product_type;
```



```
2.6
请编写一条SELECT语句，求出销售单价（sale_price 列）合计值大于进货单价（purchase_price 列）合计值1.5倍的商品种类。执行结果如下所示。

product_type | sum  | sum 
-------------+------+------
衣服         | 5000 | 3300
办公用品     |  600 | 320
```

```mysql
-- Answer:
SELECT product_type, SUM(sale_price) AS sum, SUM(purchase_price) AS sum
FROM product
GROUP BY product_type
-- 注意HAVING的书写位置在GROUP BY之后
HAVING SUM(sale_price) > SUM(purchase_price)*1.5
-- 按照商品类型降序排列结果
ORDER BY product_type DESC;
```



```
2.7
此前我们曾经使用SELECT语句选取出了product（商品）表中的全部记录。当时我们使用了ORDERBY子句来指定排列顺序，但现在已经无法记起当时如何指定的了。请根据下列执行结果，思考ORDERBY子句的内容。
```

```mysql
-- Answer:
SELECT *
FROM product
-- 由于regist_date有NULL，使用DESC时NULL会在最下方
-- ORDER BY regist_date DESC, sale_price ASC;
-- 使NULL在最上方，对整列取负(加"-"号)，ASC即可
ORDER BY -regist_date ASC, sale_price ASC;
```

