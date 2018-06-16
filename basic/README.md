# 基础知识

这里摘录了《SQL 必知必会》一书的要点。笔记的顺序是颠倒的，即书中越新的部分，越靠前。

倒序记笔记有一个好处，开头是最难的，越往后看越简单。


创建分组

```sql
SELECT vend_id, COUNT(＊) AS num_prods￼
FROM Products￼
GROUP BY vend_id;
```

组合聚集函数

```sql
SELECT 
    COUNT(＊) AS num_items,￼
    MIN(prod_price) AS price_min,￼
    MAX(prod_price) AS price_max,￼
    AVG(prod_price) AS price_avg￼
    FROM Products;
```

DISTINCT 修饰聚集函数

```sql
SELECT AVG(DISTINCT prod_price) 
AS avg_price￼
FROM Products￼
WHERE vend_id = 'DLL01';
```

`SUM()` 函数

```sql
SELECT SUM(quantity) 
AS items_ordered￼  
FROM OrderItems￼
WHERE order_num = 20005;
```

```sql
SELECT SUM(item_price * quantity)
AS total_price￼
FROM OrderItems￼
WHERE order_num = 20005;
```

`MIN()` 函数

```sql
SELECT MIN(prod_price)
AS min_price￼
FROM Products;
```

`MAX()` 函数

```sql
SELECT MAX(prod_price)
AS max_price￼
FROM Products;
```

`COUNT()` 函数

```sql
SELECT COUNT(*)
AS num_cust￼
FROM Customers;
```

```sql
SELECT COUNT(cust_email)
AS num_cust￼
FROM Customers;
```

`AVG()` 函数

```sql
SELECT AVG(prod_price)
AS avg_price￼
FROM Products;
```

SQL Server 检索2012年全部订单

```sql
SELECT order_num￼
FROM Orders￼
WHERE DATEPART(yy, order_date) = 2012;
```

Access 的代码

```sql
SELECT order_num￼
FROM Orders￼
WHERE DATEPART('yyyy', order_date) = 2012;
```

函数

```sql
SELECT 
    vend_name,
    UPPER(vend_name) AS vend_name_upcase￼
FROM Vendors￼
ORDER BY vend_name;
```

算术计算

```sql
SELECT 
    prod_id,￼
    quantity,￼
    item_price,￼
    quantity * item_price AS expanded_price￼
FROM OrderItems￼
WHERE order_num = 20008;
```

别名

```sql
SELECT RTRIM(vend_name) + ' (' + RTRIM(vend_country) + ')'￼
AS vend_title￼
FROM Vendors￼
ORDER BY vend_name;
```

去左空格 LTRIM
去右空格 RTRIM
去空格 TRIM

```sql
SELECT RTRIM(vend_name) + ' (' + RTRIM(vend_country) + ')'￼
FROM Vendors￼
ORDER BY vend_name;
```

拼接

```sql
SELECT vend_name + ' (' + vend_country + ')'￼
FROM Vendors￼
ORDER BY vend_name;
```

```sql
SELECT Concat(vend_name, ' (', vend_country, ')')￼
FROM Vendors￼
ORDER BY vend_name;
```

脱字号 `^` 表示否定

```sql
SELECT cust_contact￼
FROM Customers￼
WHERE cust_contact
LIKE '[^JM]%'￼
ORDER BY cust_contact;
```

方括号通配符 `[]`

```sql
SELECT cust_contact￼
FROM Customers￼
WHERE cust_contact
LIKE '[JM]%'￼
ORDER BY cust_contact;
```

下划线通配符 `_`，匹配单个字符。

```sql
SELECT prod_id, prod_name￼
FROM Products￼
WHERE prod_name
LIKE '__ inch teddy bear';
```

`%` 通配符，匹配任意个字符，包括零个。

```sql
SELECT prod_id, prod_name￼
FROM Products￼
WHERE prod_name LIKE 'Fish%';
```

`NOT` 操作符

```sql
SELECT prod_name￼
FROM Products￼
WHERE NOT vend_id = 'DLL01'￼
ORDER BY prod_name;
```

`IN` 操作符

```sql
SELECT prod_name, prod_price￼
FROM Products￼
WHERE vend_id 
IN ( 'DLL01', 'BRS01' )￼
ORDER BY prod_name;
```

求值顺序

```sql
SELECT prod_name, prod_price￼
FROM Products￼
WHERE (vend_id = 'DLL01' OR vend_id = 'BRS01')￼
AND prod_price >= 10;
```

`OR` 子句

```sql
SELECT prod_name, prod_price￼
FROM Products￼
WHERE vend_id = 'DLL01' 
    OR vend_id = ‘BRS01';
```

`AND` 子句

```sql
SELECT prod_id, prod_price, prod_name￼
FROM Products￼
WHERE vend_id = 'DLL01'
    AND prod_price <= 4;
```

空值检查

```sql
SELECT prod_name￼
FROM Products  
WHERE prod_price IS NULL;
```

范围值检查

```sql
SELECT prod_name, prod_price￼
FROM Products￼
WHERE prod_price 
    BETWEEN 5 AND 10;
```

过滤搜索

```sql
SELECT prod_name, prod_price￼
FROM Products￼
WHERE prod_price = 3.49;
```

`DESC` 关键字只应用到直接位于其前面的列名

```sql
SELECT prod_id, prod_price, prod_name￼
FROM Products￼
ORDER BY 
    prod_price DESC, 
    prod_name;
```

降序排列

```sql
SELECT prod_id, prod_price, prod_name￼
FROM Products￼
ORDER BY prod_price DESC;
```

除了能用列名指出排序顺序外，`ORDER BY` 还支持按相对列位置进行排序

```sql
SELECT prod_id, prod_price, prod_name￼
FROM Products￼
ORDER BY 2, 3;
```

要按多个列排序，简单指定列名，列名之间用逗号分开即可

```sql
SELECT prod_id, prod_price, prod_name￼
FROM Products￼
ORDER BY prod_price, prod_name;
```

排序

```sql
SELECT prod_name￼
FROM Products  
ORDER BY prod_name;
```

在指定一条 `ORDER BY` 子句时，应该保证它是 `SELECT` 语句中**最后一条子句**。

使用注释

```sql
SELECT prod_name    -- 这是一条注释￼    FROM Products;
```

限制结果

在 SQL Server 和 Access 中使用 SELECT 时，可以使用 `TOP` 关键字来限制最多返回多少行

```sql
SELECT TOP 5 prod_name￼
FROM Products;
```

如果你使用 MySQL、MariaDB、PostgreSQL 或者 SQLite，需要使用 `LIMIT` 子句

```sql
SELECT prod_name
FROM Products
LIMIT 5;
```

为了得到后面的5行数据，需要指定从哪儿开始以及检索的行数

```sql
SELECT prod_name
FROM Products
LIMIT 5
OFFSET 5;
```

检索不同的值

```sql
SELECT DISTINCT vend_id
FROM Products;
```

检索所有列

```sql
SELECT * FROM Products;
```

检索多列

```sql
SELECT prod_id, prod_name, prod_price
FROM Products;
```

检索单列

```sql
SELECT prod_name FROM Products;
```

标准 SQL 由 ANSI 标准委员会管理，从而称为 *ANSI SQL*。

SQL 是一种专门用来与数据库沟通的语言。

## 常用的概念

- 外键
- 主键 `primary key` 唯一标识
- 行 `row` 表中的一个记录
- 数据类型 `data type`
- 列 `column` 表中的一个字段
- 表，一种结构化的文件，可用来存储某种特定类型的数据
- 模式 `schema` 

## REF

- [SQL 必知必会][sql]，by *Ben Forta*，人民邮电出版社，2013/05/01

[sql]: https://book.douban.com/subject/24250054/