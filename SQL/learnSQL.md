# learnSQL

### SQL DEFAULT 约束
--------------

#### SQL DEFAULT 约束

default 约束用于向列中插入默认值。

CREATE TEABLE 时的 SQL DEAFAULT 约束
下面的 SQL 在"Person"表创建时在"city"列上创建 DEFAULT 约束

``` sql
create table Persons
  (
    P_Id int NOT NULL,
    LatName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255) DEFAULT 'Sandnes'
  )
```
#### ALTER TABLE时的 SQL DEFAULT 约束

当表已被创建时，如需在"City"列创建 DEAFAULT 约束，则需要使用下面的 SQL：

``` sql
ALTER TABLE Persons
ALTER City SET DEFAULT 'SANDNES'
```

#### 撤消 DEFAULT 约束

``` sql
ALTER TABLE Persons
alter City DROP DEAFAULT
```

----------------

### SQL CREATE INDEX 语句

CREATE INDEX 语句用于在表中创建索引。

在不读取整个表的情况下，索引使数据库应用程序可以更快地查找数据。

#### 索引

我们可以在表中创建索引，以便更加快速高效地查询数据。

用户无法看到索引，它们只能被用来加速搜索/查询。

**注释：**更新一个包含索引的表需要比更新一个没有索引的表花费更多的时间，这是由于索引本身也需要更新。因此，理想的做法是仅仅在常常被搜索的列上面创建索引。

#### SQL CREATE INDEX 语法

在表上创建一个简单的索引。允许使用重复的值：

``` sql
create index inde_name
  on table_name (column_name)
```

#### CREATE INDEX 实例

下面的 SQL 语句在"Person"表的"LastName"列上创建一个名为"PIndex"的索引

``` sql
create index PIndex
  on Persons (LastName)
  -- 如果希望索引不是一个列，可以在括号中列出这些列的名称，用逗号隔开
  -- on Persons (LastName, FirstName)
```

### SQL 撤消索引、撤消表以及撤消数据库

通过使用 DROP INDEX 语句， 可以轻松

### SQL AUTO INCREMENT字段

#### 用于 MySQL 的语法

``` sql
create table Persons
  (
    ID int not NULL auto_increment,
    LastName varchar(255) not null,
    FirstName varchar(255) not null,
    Address varchar(255),
    City varchar(255),
    primary key (ID)
  )
```

#### 用于 SQL Server 的语法

``` sql
create table Persons
  (
    ID int identity(1,1) primary key,
    LastName varchar(255) not null,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255)
  )
```

### SQL 视图(Views)

视图是可视化的表。

#### SQL CREATE VIEW 语句（需要网上查询深入研究）

在 SQL 中，视图是基于 SQL 结果集的可视化的表。

**注释：**视图总是显示最新的数据！每当用户查询视图时，数据库引擎通过使用视图的 SQL 语句重建数据。

视图"Current Product List"会从"Products"表列出所有正在使用的产品（未停产的产品）。这个视图使用如下 SQL 创建：

``` sql
create view [Current Product List] as
  select productID, peoductName
  from Products
  where Discontinued=No
```

可以通过下面的 SQL 语句查询上面这个视图

`select * from [Current Product List]`

#### SQL 更新视图

- SQL create or replace view 语法

``` sql
create view [Current Product List] as
  select productID,productName,category
  from Products
  where Discontinued=No
```

#### SQL 撤消视图

``` sql
DROP VIEW view_name
```

#### 视图的作用

1. 视图隐藏了底层的表结构，简化了数据访问操作，客户端不再需要知道底层结构及其之间的关系。
2. 视图提供了一个统一访问数据的接口。（即允许用户通过视图访问数据的安全机制，而不授予用户直接访问底层表的权限）
3. 从而加强了安全性，使用户只能看到视图所显示的数据。
4. 视图还可以被嵌套，一个视图可以嵌套另一个视图。

### SQL NULL 值

NULL 值得处理方式与其他值不同。

NULL 用作未知得或不适用的值的占位符。

**注释：**无法比较 NULL 和 0;它们时不等价的。

#### SQL 的 NULL 值处理

无法使用比较运算符来测试 NULL 值，比如 =、< 或 <>。

**必须使用 IS　NULL　和　IS　NOT　NULL　操作符。**

``` sql
select lastName,fistName,Address from Persons
where Address IS NULL
```

### [SQL 通用数据类型](http://www.runoob.com/sql/sql-datatypes-general.html)

### SQL 函数

#### SQL Aggregate 函数

SQL Aggregate 函数计算从列中取得的值，返回一个单一的值。

有用的 Aggregate 函数：

- AVG(): 返回平均值
- COUNT(): 返回行数
- FIRST(): 返回第一个记录的值
- LAST(): 返回最后一个记录的值
- MAX(): 返回最大值
- MIN(): 返回最小值
- SUM(): 返回总和

#### SQL Scalar 函数

SQL Scalar 函数基于某个输入值，返回一个单一的值。

有用的 Scalar 函数

- UCASE(): 将某个字段转换为大写
- LCASE(): 将某个字段转换为小写
- MID(): 将某个文本字段提取字符，MySql中使用
- SubString(字段， 1， end): 从某个文本字段提取字符
- LEN(): 返回某个文本字段的长度
- ROUND(): 对某个数值字段进行指定小数位数的四舍五入
- NOW(): 返回当前的系统日期和时间
- FORMAT(): 格式化某个字段的显示方式
