# SQL 基础概念

## 基础概念

- 数据库：保存有组织的数据的容器。
- 常见误区：用户使用的软件被称为数据库管理系统（DBMS），数据库是通过 DBMS 创建和操纵的容器。
- 表：特定类型数据的结构化清单。
- schema：元数据信息，关于数据库和表的布局及特性的信息。
- 列：表中的一个字段。所有表都是由一个或多个列组成的。
- 行：表中的一个记录。
- 主键（primary key）：一列（或几列），其值能够唯一标识表中每一行。

### 主键应该满足的条件

- 任意两行都不具有相同的主键值；
- 每一行都必须具有一个主键值（主键列不允许空值 NULL）；
- 主键列中的值不允许修改或更新；
- 主键值不能重用（如果某行从表中删除，它的主键不能赋给以后的新行）。

> 注意：尽管现在的数据库允许通过 `UPDATE` 修改主键，但是担心有外键约束，最好的使用方式是创建一个新的记录，并将旧记录标记为失效状态。

## 查询语法

### DISTINCT

对结果集进行去重。

```sql
SELECT DISTINCT vend_id FROM products;
```

### LIMIT

限制结果集数量。

```sql
SELECT prod_name FROM products LIMIT 5;
```

### OFFSET

偏移量。

```sql
SELECT prod_name FROM products OFFSET 5;
```

OFFSET 和 LIMIT 位置可以互换，下面二者结果相同：

```sql
SELECT prod_name FROM products OFFSET 5 LIMIT 3;
SELECT prod_name FROM products LIMIT 3 OFFSET 5;
```

### ORDER BY

排序，可以通过被选择列或者非被选择列进行排序。

```sql
SELECT prod_id, prod_price, prod_name
FROM Products
ORDER BY prod_price, prod_name;
```

还有一种按照选择列的位置进行排序：

```sql
SELECT prod_id, prod_price, prod_name
FROM Products
ORDER BY 2, 3;
```

注意，使用位置排序时就无法使用非选择列了。默认升序，可用 `DESC` 指定降序。

### WHERE

过滤条件。注意 `ORDER BY` 应该在 `WHERE` 子句之后。常见操作符： `=`, `!=`, `>`, `>=`, `<`, `<=`, `!>`, `!<`。

- BETWEEN a AND b 用来过滤 a 和 b 之间的值。
- IS NULL 空值检查。

### AND / OR

组合过滤条件。任何时候使用具有 AND 和 OR 操作符的 WHERE 子句，都应该使用圆括号明确地分组操作符。不要过分依赖默认求值顺序。

### IN

IN 操作符用来指定条件范围，是 OR 操作符的一种简便写法：

```sql
SELECT prod_name, prod_price FROM Products WHERE vend_id IN ('DLL01','BRS01') ORDER BY prod_name;

SELECT prod_name, prod_price FROM Products WHERE vend_id = 'DLL01' OR vend_id = 'BRS01' ORDER BY prod_name;
```

### NOT

否认其后的过滤条件。

### LIKE 与通配符

- `%`：匹配任意数量字符。
- `_`：匹配单个字符。
- `[]`：匹配字符集中的一个字符。
- `[^]`：不匹配字符集中的字符。

### 计算字段

例如多个列之间的内容进行拼接，视 DBMS 不同可以使用 `+`、`||` 操作符或者 `CONCAT` 函数进行处理。还有 `TRIM`、`LTRIM`、`RTRIM` 等用来去掉空格。

### 别名 AS

为计算字段或者某些字段赋予一个新的名字，且可以在子句中使用。例如别名可用于排序。

### COUNT 函数

- 使用 `COUNT(*)` 计数表中的行。
- 使用 `COUNT(column)` 对特定列中具有值的行进行计数（会忽略 NULL 值）。

### GROUP BY

对数据进行分组，分组后 `COUNT`, `SUM`, `MAX` 等聚集函数就可以在每个组中生效。`NULL` 也会被分为一个组。

> 注意：`GROUP BY` 出现在 `WHERE` 子句之后，`ORDER BY` 子句之前。

### HAVING

`HAVING` 和 `WHERE` 在使用上类似，不同点在于：

- `WHERE` 对行进行过滤；
- `HAVING` 对分组进行过滤。

### 子查询

子查询可以作为父查询中的 `WHERE` 条件或作为检索项：

```sql
SELECT cust_name, cust_contact
FROM Customers
WHERE cust_id IN (
  SELECT cust_id FROM Orders WHERE order_num IN (
    SELECT order_num FROM OrderItems WHERE prod_id = 'RGAN01'
  )
);

SELECT cust_name, cust_state,
  (
    SELECT COUNT(*) FROM Orders WHERE Orders.cust_id = Customers.cust_id
  ) AS orders
FROM Customers
ORDER BY cust_name;
```

> 注意：子查询作为父查询的计算字段时不如 `JOIN` 直观。子查询通常只能返回一列。

## 联结（JOIN）

### 自联结

示例（使用子查询方式）：

```sql
SELECT cust_id, cust_name, cust_contact
FROM Customers
WHERE cust_name = (
  SELECT cust_name FROM Customers WHERE cust_contact = 'Jim Jones'
);
```

使用自联结和别名通常更高效：

```sql
SELECT c1.cust_id, c1.cust_name, c1.cust_contact
FROM Customers AS c1, Customers AS c2
WHERE c1.cust_name = c2.cust_name
  AND c2.cust_contact = 'Jim Jones';
```

### 内联结

返回两表匹配成功的行（INNER JOIN）。

### 外联结

例如左外联结会包含匹配成功的行，左表中未与右表匹配成功的行也会存在，但是右表中相关字段为空值（LEFT JOIN）。

## 插入（INSERT）

```sql
INSERT [INTO] tablename[(columns)] VALUES (...);
INSERT [INTO] tablename[(columns)] SELECT columns FROM tablename;
```

整表复制语法示例：

```sql
CREATE TABLE CustCopy AS SELECT * FROM Customers;
-- 或者（某些DBMS）
SELECT * INTO CustCopy FROM Customers;
```

## 更新（UPDATE）

```sql
UPDATE Customers
SET cust_contact = 'Sam Roberts',
    cust_email = 'sam@toyland.com'
WHERE cust_id = 1000000006;
```

## 删除（DELETE）

```sql
DELETE FROM Customers WHERE cust_id = 1000000006;
-- 直接清空表的所有内容（某些 DBMS 支持）
TRUNCATE TABLE tablename;
```

## 表格操作

### 创建表格

```sql
CREATE TABLE products_aaa (
  prod_id     CHAR(10) NOT NULL,
  vend_id     CHAR(10) NOT NULL,
  prod_name   CHAR(10) NOT NULL,
  prod_price  DECIMAL(8,2) NOT NULL DEFAULT 1.23,
  prod_desc   VARCHAR(1000) NULL
);
```

### 修改表格

```sql
ALTER TABLE Vendors ADD vend_phone CHAR(20);
```

### 重命名表

查询 DBMS 相关文档，不同 DBMS 操作可能不同。

## 视图（VIEW）

视图为虚拟的表。它们包含的不是数据而是根据需要检索数据的查询。视图提供了一种封装 SELECT 语句的层次，可用来简化数据处理，重新格式化或保护基础数据。

### 视图创建示例

```sql
CREATE VIEW ProductCustomers AS
SELECT cust_name, cust_contact, prod_id
FROM Customers, Orders, OrderItems
WHERE Customers.cust_id = Orders.cust_id
  AND OrderItems.order_num = Orders.order_num;
```

使用视图重新格式化检索出的数据示例：

直接写法：

```sql
SELECT RTRIM(vend_name) || ' (' || RTRIM(vend_country) || ')' AS vend_title
FROM Vendors
ORDER BY vend_name;
```

视图写法：

```sql
CREATE VIEW VendorLocations AS
SELECT RTRIM(vend_name) || ' (' || RTRIM(vend_country) || ')' AS vend_title
FROM Vendors;

SELECT vend_title FROM VendorLocations;
```

## 存储过程

存储过程就是为以后使用而保存的一条或多条 SQL 语句。可将其视为批文件，虽然它们的作用不仅限于批处理。存储过程很强大，但是其可能并不安全，可在需要时再学习。

## 事务（TRANSACTION）

- 事务（transaction）指一组 SQL 语句；
- 回退（rollback）指撤销指定 SQL 语句的过程；
- 提交（commit）指将未存储的 SQL 语句结果写入数据库表；
- 保留点（savepoint）指事务处理中设置的临时占位符，可以对它发布回退（与回退整个事务处理不同）。

一般示例写法：

```sql
BEGIN;
SAVE TRANSACTION point;
-- IF ERROR ROLLBACK TRANSACTION point;
COMMIT;
```

具体写法请参考相关 DBMS 文档。

## 游标

cursor 可以按需要学习。

## 约束（CONSTRAINTS）

### 主键约束（PRIMARY KEY）

创建表时定义示例：

```sql
CREATE TABLE Vendors (
  vend_id   CHAR(10) NOT NULL PRIMARY KEY,
  vend_name CHAR(50) NOT NULL,
  vend_address CHAR(50) NULL,
  vend_city CHAR(50) NULL,
  vend_state CHAR(5) NULL,
  vend_zip CHAR(10) NULL,
  vend_country CHAR(50) NULL
);
```

修改表时定义：

```sql
ALTER TABLE Vendors ADD CONSTRAINT PRIMARY KEY (vend_id);
```

注意：有些 DBMS 只允许在创建表时定义主键。

### 外键（FOREIGN KEY）

创建表时定义示例：

```sql
CREATE TABLE Orders (
  order_num INTEGER NOT NULL PRIMARY KEY,
  order_date DATETIME NOT NULL,
  cust_id CHAR(10) NOT NULL REFERENCES Customers(cust_id)
);
```

修改表时定义：

```sql
ALTER TABLE Orders
ADD CONSTRAINT FOREIGN KEY (cust_id) REFERENCES Customers (cust_id);
```

外键有助防止意外删除，定义外键后，外键不允许删除在另外一个表中具有关联行的行。但注意不要启动级联删除，否则可能会因为删除某条记录导致大量数据被意外级联删除。

工程上常用软删除：添加一个布尔字段 `is_delete` 或使用日期字段记录删除时间，未删除的行此字段为 `NULL`。

### 唯一约束（UNIQUE）

类似于主键，但区别包括：

- 表可包含多个唯一约束，但每个表只允许一个主键；
- 唯一约束列可包含 NULL 值；
- 唯一约束列可修改或更新；
- 唯一约束列的值可重复使用；
- 唯一约束不能用来定义外键。

### 检查约束（CHECK）

检查约束用来保证一列（或一组列）中的数据满足指定条件。常见用途：

- 检查最小或最大值；
- 指定范围；
- 只允许特定的值（例如性别字段只允许 `M` 或 `F`）。

示例：

```sql
CREATE TABLE OrderItems (
  order_num INTEGER NOT NULL,
  order_item CHAR(10) NOT NULL,
  prod_id INTEGER NOT NULL,
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  item_price MONEY NOT NULL
);
```

## 索引（INDEX）

索引注意点：

- 索引改善检索操作的性能，但降低了数据插入、修改和删除的性能；
- 索引数据可能占用大量存储空间；
- 并非所有数据都适合做索引（例如取值少的数据如州不适合）；
- 索引用于数据过滤和排序；
- 可以在索引中定义多个列（例如州+城市），只有在以该顺序排序时才有用。

创建索引示例：

```sql
CREATE INDEX prod_name_ind ON Products (prod_name);
```

## 触发器（TRIGGER）

创建触发器示例：

```sql
CREATE TRIGGER customer_state
AFTER INSERT OR UPDATE
FOR EACH ROW
BEGIN
  UPDATE Customers
  SET cust_state = UPPER(cust_state)
  WHERE Customers.cust_id = :OLD.cust_id;
END;
```

触发器的创建方式依赖于具体 DBMS，需参考相应文档。
