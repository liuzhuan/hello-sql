# 基础知识

这里摘录了《SQL 必知必会》一书的要点。笔记的顺序是颠倒的，即书中越靠后的部分，在笔记中越靠前。

倒序记笔记有一个好处，开头是最难的，越往后看越简单。

## 使用的数据表

本书使用的表是一个玩具经销商的订单录入系统的组成部分，表中数据包含以下内容：

- 供应商（Vendors 表）
- 产品目录（Products 表）
- 顾客信息（Customers 表）
- 顾客订单（Orders 表和 OrderItems 表）

各表的字段说明如下：

Vendors 表

```
vend_id         唯一ID（主键）
vend_name       名称
vend_address    地址
vend_city       所在城市
vend_state      所在州
vend_zip        邮政编码
vend_country    所在国家
```

Products 表

```
prod_id     唯一产品 ID（主键）
vend_id     产品供应商 ID（外键，关联到 Vendors 的 vend_id 列）
prod_name   产品名
prod_price  产品价格
prod_desc   产品描述
```

Customers 表

```
cust_id     唯一的顾客ID（主键）
cust_name
cust_address
cust_city
cust_state
cust_zip
cust_country
cust_contact
cust_email
```

Orders 表

```
order_num   唯一的订单号（主键）
order_date  订单日期
cust_id     订单顾客ID（外键，关联到 Customers 表的 cust_id 列）
```

OrderItems 表

```
order_num   订单号（主键之一，兼外键，关联到 Orders 表的 order_num 列）
order_item  订单物品号（主键之二，订单内的顺序）
prod_id     产品ID（外键，关联到 Products 表的 prod_id）
quantity    物品数量
item_price  物品价格
```

## 重命名表

每个 DBMS 对于重命名的支持有所不同。

## 删除表

```sql
DROP TABLE CustCopy;
```

许多 DBMS 允许强制实施有关规则，防止删除与其他表相关联的表。

## 更新表

更新表定义，可以使用 `ALTER TABLE` 语句。

例如，给一个表中增加一个列：

```sql
ALTER TABLE Vendors
ADD vend_phone CHAR(20);
```

删除列如下：

```sql
ALTER TABLE Vendors
DROP COLUMN vend_phone;
```

## 创建表

可以使用 `CREATE TABLE` 创建表。

```sql
CREATE TABLE Products(
    prod_id     CHAR(10)        NOT NULL,
    vend_id     CHAR(10)        NOT NULL,
    prod_name   CHAR(254)       NOT NULL,
    prod_price  DECIMAL(8,2)    NOT NULL,
    prod_desc   VARCHAR(1000)   NULL
);
```

### 指定默认值

默认值在 `CREATE TABLE` 语句的列定义中用关键字 `DEFAULT` 指定。

```sql
CREATE TABLE OrderItems (
    order_num   INTEGER         NOT NULL,
    order_item  INTEGER         NOT NULL,
    prod_id     CHAR(10)        NOT NULL,
    quantity    INTEGER         NOT NULL    DEFAULT 1,
    item_price  DECIMAL(8,2)    NOT NULL
)
```

默认值经常用于日期或时间戳列。例如通过指定引用系统日期的函数或变量，将系统时间用作默认日期。

例如，在 MySQL 中获取系统时间的变量是 `CURRENT_DATE()`；SQLite 中为 `date('now')`。

## 删除数据

```sql
DELETE FROM Customers
WHERE cust_id = '1000000006';
```

使用外键确保引用完整性的一个好处是，DBMS 通常可以防止删除某个关系需要用到的行。

## 更新数据

比如，更新客户 1000000005 的电子邮件：

```sql
UPDATE Customers
SET cust_email = 'kim@thetoystore.com'
WHERE cust_id = '1000000005';
```

更新多个列的语法稍有不同：

```sql
UPDATE Customers
SET
    cust_contact = 'Sam Roberts',
    cust_email = 'sam@toyland.com'
WHERE cust_id = '1000000006';
```

要删除某个列的值，可设置它为 NULL：

```sql
UPDATE Customers
SET cust_email = NULL
WHERE cust_id = '1000000005';
```

## 从一个表复制到另一个表

要将一个表的内容复制到一个全新的表，可以使用 `SELECT INTO` 语句。

```sql
SELECT *
INTO CustCopy
FROM Customers;

-- MariaDB, MySQL, Oracle, PostgreSQL 和 SQLite 语法稍有不同
CREATE TABLE CustCopy AS
    SELECT *
    FROM Customers;
```

## 数据插入

### 插入完整的行

```sql
INSERT INTO Customers
VALUES(
    '1000000006',
    'Toy Land',
    '123 Any Street',
    'New York',
    'NY',
    '11111',
    'USA',
    NULL,
    NULL
);
```

虽然这种语法简单，但并不安全，应该尽量避免使用。因为它高度依赖于表中列的定义次序。

⚠️ 注意，编写完全依赖于特定次序的 SQL 语句是很不安全的，迟早会出问题。

推荐的做法是：

```sql
INSERT INTO Customers(
    cust_id,
    cust_name,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country,
    cust_contact,
    cust_email
)
VALUES(
    '1000000006',
    'Toy Land',
    '123 Any Street',
    'New York',
    'NY',
    '11111',
    'USA',
    NULL,
    NULL
)
```

给出列名使 SQL 代码可以有更通用的适用范围，即使表结构发生了变化。

### 插入部分行

对于可以为 NULL 的列，列名可以省略，比如：

```sql
INSERT INTO Customers(
    cust_id,
    cust_name,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country
)
VALUES(
    '1000000006',
    'Toy Land',
    '123 Any Street',
    'New York',
    'NY',
    '11111',
    'USA'
)
```

### 插入检索出的数据

```sql
INSERT INTO Customers(
    cust_id,
    cust_contact,
    cust_email,
    cust_name,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country
)
SELECT
    cust_id,
    cust_contact,
    cust_email,
    cust_name,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country
FROM CustNew;
```

DBMS 一点儿也不关心 SELECT 返回的列名。它使用的使列的位置。

## 创建组合查询

可以使用 `UNION` 操作符来组合数条 SQL 查询。

假如需要 Illinois, Indiana 和 Michigan 等美国几个州的所有顾客的报表，还想包括不管位于哪个州的所有的 Fun4All。首先，使用单条 SQL 语句如下：

```sql
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_state IN ('IL', 'IN', 'MI');

SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_name = 'Fun4All';
```

组合这两条语句，如下：

```sql
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_state IN ('IL', 'IN', 'MI')
UNION
SELECT cust_name, cust_contact, cust_email
FROM Customers
WHERE cust_name = 'Fun4All';
```

使用 `UNION` 时，重复的行会被自动取消。这是 `UNION` 的默认行为，如果愿意也可以改变它，使用 `UNION ALL` 代替 `UNION` 即可。

### 对组合查询结果排序

在用 `UNION` 组合查询时，只能使用一种 `ORDER BY` 子句，它必须位于最后一条 `SELECT` 语句之后。

## 组合查询

SQL 允许执行多个查询，并将结果作为一个查询结果集返回。这些组合查询通常成为并（union）或复合查询（compound query）。

## 使用带聚集函数的联结

比如，要检索所有顾客及每个顾客所下的订单数：

```sql
SELECT 
    Customers.cust_id,
    COUNT(Orders.order_num) AS num_ord
FROM Customers INNER JOIN Orders
    ON Customers.cust_id = Orders.cust_id
GROUP BY Customers.cust_id;
```

## 使用不同类型的联结

除了内联结或等值联结外，还有其他三种联结：

- 自联结（self join）
- 自然联结（natural join）
- 外联结（outer join）

### 自联结

如果要给与 Jim Jones 同一公司的所有顾客发送一封邮件。

```sql
SELECT c1.cust_id, c1.cust_name, c1.cust_contact
FROM Customers AS c1, Customers AS c2
WHERE
    c1.cust_id = c2.cust_id
    AND c2.cust_name = 'Jim Jones'
```

### 自然联结

自然联结排除多次出现的列，使每一列只返回一次。

```sql
SELECT C.*, O.order_num, O.order_date, OI.prod_id, OI.quantity, OI.item_price
FROM Customers AS C, Orders AS O, OrderItems as OI
WHERE 
    C.cust_id = O.cust_id
    AND OI.order_num = O.order_num
    AND prod_id = 'RGAN01';
```

### 外联结

外联结包含了那些在相关表中没有关联行的行。

比如，要检索包括没有订单顾客在内的所有顾客的订单：

```sql
SELECT Customers.cust_id, Orders.order_num
FROM Customers LEFT OUTER JOIN Orders
    ON Customers.cust_id = Orders.cust_id;
```

在使用 `OUTER JOIN` 语法时，必须使用 `RIGHT` 或 `LEFT` 关键字指定包含所有行的表。

上面的例子使用 `LEFT OUTER JOIN` 从 `FROM` 子句左边的表（Customers）中选择所有行。

## 使用表别名

```sql
SELECT cust_name, cust_contact
FROM Customers AS C, Orders AS O, OrderItems AS OI
WHERE
    C.cust_id = O.cust_id
    AND OI.order_num = O.order_num
    AND prod_id = 'RGAN01';
```

## 创建联结

```sql
SELECT vend_name, prod_name, prod_price
FROM Vendors, Products
WHERE Vendors.vend_id = Products.vend_id; -- 必须使用完全限定列名
```

### WHERE 子句的重要性

在一条 SELECT 语句中联结多个表时，相应的关系是在运行时构造的。

没有 SELECT 子句，第一个表的每一行将与第二个表中的每一行配对，而不管它们逻辑上是否能配在一起。此时返回的结果也称作 **笛卡尔积**（cartesian product）。

有时候，返回笛卡尔积的联结，也称作**叉联结**（cross join）。

### 内联结

等值联结（equi-join）基于两个表之间的相等测试，也称为内联结（inner join）。

也可以使用不同的语法，明确联结类型。

```sql
SELECT vend_name, prod_name, prod_price
FROM Vendors INNER JOIN Products
    ON Vendors.vend_id = Products.vend_id;
```

此时，联结条件使用 `ON` 子句而不是 `WHERE` 子句给出。

ANSI SQL 规范首选 `INNER JOIN` 语法。

### 联结多个表

```sql
SELECT prod_name, vend_name, prod_price, quantity
FROM OrderItems, Products, Vendors
WHERE
    Products.vend_id = Vendors.vend_id
    AND OrderItems.prod_id = Products.prod_id
    AND order_num = 20007;
```

⚠️ 注意，不要联结不必要的表，联结的表越多，性能下降越厉害。

```sql
SELECT cust_name, cust_contact
FROM Customers, Orders, OrderItems
WHERE
    Customers.cust_id = Orders.cust_id
    AND OrderItems.order_num = Orders.order_num
    AND prod_id = 'RGAN01';
```

## 联结

联结（join）是利用 SQL 的 SELECT 能执行的最重要的操作，理解联结及其语法是学习 SQL 极为重要的部分。

要理解联结，必需了解关系表和关系数据库的一些基础知识。

### 关系表

关系数据库设计的基础是，相同的数据出现多次绝不是一件好事。关系表的设计就是要把信息分解成多个表，一类数据一个表。

关系数据库的可伸缩性（scalable）远比非关系数据库要好。

### 为什么使用联结

可伸缩性虽然有好处，但是需要付出代价，在多个表中才能找到所有的数据。

联结是一种机制，用来在一条 SELECT 语句中将多个表关联。联结在运行时关联表中正确的行。

## 作为计算字段使用子查询

使用子查询的另一个方法是创建计算字段。假如需要显示 Customers 表中每个顾客的订单总数。所需步骤如下：

1. 从 Customers 表中检索顾客列表
2. 对于每个顾客，统计其在 Orders 表中的订单数目

首先，统计单个顾客（比如 1000000001）的订单总数如下：

```sql
SELECT COUNT(*) AS orders
FROM Orders
WHERE cust_id = '1000000001';
```

要查询每个顾客的订单总量，可以这么处理：

```sql
SELECT 
    cust_name,
    cust_state,
    (
        SELECT COUNT(*)
        FROM Orders
        WHERE Orders.cust_id = Customers.cust_id -- 使用完全限定列名
    ) AS orders
FROM Customers
ORDER BY cust_name;
```

## 利用子查询进行过滤

如果要检索订购物品 GRAN01 的所有顾客，首先列出三个子查询：

```sql
SELECT order_num
FROM OrderItems
WHERE prod_id = 'RGAN01'; -- => 20007, 20008

SELECT cust_id
FROM Orders
WHERE order_num IN (20007, 20008); -- => 1000000004, 1000000005
```

以上两个子查询可以合并为：

```sql
SELECT cust_id
FROM Orders
WHERE order_num IN (
    SELECT order_num
    FROM OrderItems
    WHERE prod_id = 'RGAN01'
);
```

如果要查询顾客信息，可以进一步嵌套子查询：

```sql
SELECT cust_name, cust_contact
FROM Customers
WHERE cust_id IN (
    SELECT cust_id
    FROM Orders
    WHERE order_num IN (
        SELECT order_num
        FROM OrderItems
        WHERE prod_id = 'RGAN01'
    )
);
```

⚠️ 注意，作为子查询的 SELECT 语句只能查询单个列。

## 子查询

子查询（subquery）指嵌套在其他查询中的查询。

## SELECT 子句顺序

```sql
SELECT
FROM
WHERE
GROUP BY
HAVING
ORDER BY
```

## 分组和排序

一般在使用 `GROUP BY` 子句时，应该也给出 `ORDER BY` 子句。这是保证数据正确排序的唯一方法。

```sql
SELECT order_num, COUNT(*) AS items
FROM OrderItems
GROUP BY order_num
HAVING COUNT(*) >= 3
ORDER BY items, order_num;
```

## 过滤分组

`WHERE` 过滤行，而 `HAVING` 过滤分组。两者职责类似。

```sql
-- 列出拥有两个或更多订单的顾客ID
SELECT cust_id, COUNT(*) AS orders
FROM Orders
GROUP BY cust_id
HAVING COUNT(*) >= 2;
```

`WHERE` 在数据分组前进行过滤，`HAVING` 在数据分组后进行过滤。

```sql
-- 列出具有两个以上产品且其价格大于等于4的供应商
SELECT vent_id, COUNT(*) AS num_prods
FROM Products
WHERE prod_price >= 4
GROUP BY vent_id
HAVING COUNT(*) >= 2;
```

## 创建分组

`GROUP BY` 分组的重要规定

- `GROUP BY` 子句可以包含任意数目的列，可以嵌套分组
- 如果嵌套分组，则在最后指定的分组上进行汇总
- `GROUP BY` 子句的每一列都必须是检索列或有效表达式，不能使用别名
- 大部分 SQL 实现不允许长度可变的数据类型（如文本或备注型字段）
- `GROUP BY` 子句必须出现在 `WHERE` 子句之后，`ORDER BY` 子句之前

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
SELECT MIN(prod_price) AS min_price￼
FROM Products;
```

`MAX()` 函数

```sql
SELECT MAX(prod_price) AS max_price￼
FROM Products;
```

`COUNT()` 函数

```sql
SELECT COUNT(*) AS num_cust￼
FROM Customers;
```

```sql
SELECT COUNT(cust_email) AS num_cust￼
FROM Customers;
```

`AVG()` 函数

```sql
SELECT AVG(prod_price) AS avg_price￼
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

## 别名

```sql
SELECT RTRIM(vend_name) + ' (' + RTRIM(vend_country) + ')'￼
    AS vend_title￼
FROM Vendors￼
ORDER BY vend_name;
```

去左空格 `LTRIM`；去右空格 `RTRIM`；去空格 `TRIM`。

```sql
SELECT RTRIM(vend_name) + ' (' + RTRIM(vend_country) + ')'￼
FROM Vendors￼
ORDER BY vend_name;
```

## 拼接

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
WHERE cust_contact LIKE '[^JM]%'￼
ORDER BY cust_contact;
```

方括号通配符 `[]`

```sql
SELECT cust_contact￼
FROM Customers￼
WHERE cust_contact LIKE '[JM]%'￼
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
WHERE 
    (vend_id = 'DLL01' OR vend_id = 'BRS01')￼
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
- [Join (SQL)][wiki-join]

[sql]: https://book.douban.com/subject/24250054/
[wiki-join]: https://en.wikipedia.org/wiki/Join_(SQL)