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

```
Answer:
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

