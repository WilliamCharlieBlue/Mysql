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

### 6.2 .1 专用窗口函数

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

