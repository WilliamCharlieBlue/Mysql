# 02 Mysql Database Basic

### 2.1 Mysql 基础知识

创建和删除DATABASE

```mysql
CREATE DATABASE shop;
DROP DATABASE shop;
```

TABLE的参数

```shell
INTEGER	整型
DATE	日期
CHAR(len)	定长字符(len可选择设定)
VARCHAR(len)	可变长度字符(len可选择设定)
NOT NULL	非空
PRIMARY KEY	主键
```

创建TABLE实例

```mysql
CREATE TABLE product
(product_id CHAR(4) NOT NULL,
 product_name VARCHAR(100) NOT NULL,
 product_type VARCHAR(32) NOT NULL,
 sale_price INTEGER ,
 purchase_price INTEGER ,
 regist_date DATE ,
 PRIMARY KEY (product_id));
```

增加和删除列

```mysql
ALTER TABLE product ADD COLUMN product_name_pinyin VARCHAR(100);
ALTER TABLE product DROP COLUMN product_name_pinyin;
```

更新product中'厨房用具'对应的sale_price和purchase_price

```mysql
UPDATE product
   SET sale_price = sale_price * 10,
       purchase_price = purchase_price / 2
 WHERE product_type = '厨房用具';  
```

插入数据

```mysql
/* 单行插入 */
INSERT INTO ProductIns (product_id, product_name, product_type, sale_price, purchase_price, regist_date) VALUES ('0006', '叉子', '厨房用具', 500, NULL, '2009-09-20'); 
/* 省略列清单 */
INSERT INTO ProductIns VALUES ('0002', '打孔器','办公用品', 500, 320, '2009-09-11');
/* 多行插入使用ALL，DUAL是Oracle特有（安装时的必选项） */
/* “SELECT *FROM DUAL” 部分也只是临时性的，并没有实际意义。*/
INSERT ALL INTO ProductIns VALUES ('0002', '打孔器', '办公用品', 500, 320, '2009-09-11')
INTO ProductIns VALUES ('0003', '运动T恤', '衣服', 4000, 2800, NULL)
INTO ProductIns VALUES ('0004', '菜刀', '厨房用具', 3000, 2800, '2009-09-20')
SELECT * FROM DUAL; 
```

从Product使用SELECT复制数据到ProductCopy 

```mysql
INSERT INTO ProductCopy (product_id, product_name, product_type, sale_price, purchase_price, regist_date)
SELECT product_id, product_name, product_type, sale_price, purchase_price, regist_date
FROM Product; 
```



### 2.2 Practice

```
1.1 
编写一条 CREATE TABLE 语句，用来创建一个包含表 1-A 中所列各项的表 Addressbook （地址簿），并为 regist_no （注册编号）列设置主键约束
1.2 
假设在创建练习1.1中的 Addressbook 表时忘记添加如下一列 postal_code （邮政编码）了，请把此列添加到 Addressbook 表中。
1.3 
编写 SQL 语句来删除 Addressbook 表。
1.4
编写 SQL 语句来恢复删除掉的 Addressbook 表。
```

![image-20201215232840229](https://cdn.jsdelivr.net/gh/WilliamCharlieBlue/img@main/img/image-20201215232840229.png)