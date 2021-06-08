# 1. 使用MySQL

## 1.1 登录数据库
`mysql -h 127.0.0.1 -u root -p****`
- -h指连接的主机名，因为连接的本机，所以为localhost；-
- -u表示用户名，此处为root；
- -p指用户密码，可以在-p后面紧接着输入密码。

```mysql
mysql> exit # 退出 使用 “quit;” 或 “\q;” 一样的效果
mysql> status;  # 显示当前mysql的version的各种信息
mysql> select version(); # 显示当前mysql的version信息
mysql> show global variables like 'port'; # 查看MySQL端口号
```
## 1.2 通过cmd管理员模式开启关闭MySQL服务
`net stop mysql` 关闭服务

`net start mysql` 开启服务

## 1.3 修改root用户密码
- 方法1： 用`SET PASSWORD`命令
    - 首先登录`MySQL`。 
    - 格式：`mysql> set password for 用户名@localhost = password('新密码');`
    - 例子：`mysql> set password for root@localhost = password('123');`
- 方法2：用`mysqladmin`
    - 格式：`mysqladmin -u用户名 -p旧密码 password 新密码 `
    - 例子：`mysqladmin -uroot -p123456 password 123` 
- 方法3：在忘记root密码的时候
> 在忘记root密码的时候，可以这样 
>
> 以windows为例： 
>
> 关闭正在运行的MySQL服务。 
>
> 打开DOS窗口，转到mysql\bin目录。 
>
> 输入mysqld --skip-grant-tables 回车。--skip-grant-tables 的意思是启动MySQL服务的时候跳过权限表认证。 
>
> 再开一个DOS窗口（因为刚才那个DOS窗口已经不能动了），转到mysql\bin目录。 
>
> 输入mysql回车，如果成功，将出现MySQL提示符 >。 
>
> 连接权限数据库： use mysql; 。 
>
> 改密码：update user set password=password("123") where user="root";（别忘了最后加分号） 。 
>
> 刷新权限（必须步骤）：flush privileges;　。 
>
> 退出 quit。 
>
> 注销系统，再进入，使用用户名root和刚才设置的新密码123登录。

# 2. 数据库基本操作

- 创建数据库 `CREATE DATABASE 数据库名;`
- 删除数据库 `DROP DATABASE 数据库名;`
- 查看已存在数据库 `SHOW DATABASES;`
- 选择数据库 `USE 数据库名;`
- 导入SQL脚本`mysql> source D:\populate.sql` 

# 3. 表操作

## 3.1 创建表

> CREATE TABLE 表名(
>
> 属性名 数据类型 [完整性约束条件]，
>
> 
>
> 属性名 数据类型 [完整性约束条件]，
>
> 属性名 数据类型 [完整性约束条件]，
>
> );

```mysql
-- 如果数据库中存在user_accounts表，就把它从数据库中drop掉
DROP TABLE IF EXISTS `user_accounts`;
CREATE TABLE `user_accounts` (
  `id`             int(100) unsigned NOT NULL AUTO_INCREMENT primary key,
  `password`       varchar(32)       NOT NULL DEFAULT '' COMMENT '用户密码',
  `reset_password` tinyint(32)       NOT NULL DEFAULT 0 COMMENT '用户类型：0－不需要重置密码；1-需要重置密码',
  `mobile`         varchar(20)       NOT NULL DEFAULT '' COMMENT '手机',
  `create_at`      timestamp(6)      NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  `update_at`      timestamp(6)      NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
  -- 创建唯一索引，不允许重复
  UNIQUE INDEX idx_user_mobile(`mobile`)
)
ENGINE=InnoDB DEFAULT CHARSET=utf8
COMMENT='用户表信息';
```

数据类型的属性解释

- `NULL`：数据列可包含NULL值；
- `NOT NULL`：数据列不允许包含NULL值；
- `DEFAULT`：默认值；
- `PRIMARY KEY`：主键；
- `AUTO_INCREMENT`：自动递增，适用于整数类型；
- `UNSIGNED`：是指数值类型只能为正数；
- `CHARACTER SET name`：指定一个字符集；
- `COMMENT`：对表或者字段说明；
- `ENGINE=InnoDB`设置引擎为`InnoDB`或`MyISAM`
- `DEFAULT CHARSET=utf8`设置字符集



**多字段主键**

- PRIMARY KEY(属性名1，属性名2，……，属性名n)

- > CREATE TABLE example2(
  >
  >   		 stu_id INT,
  >						
  >   		 course_id INT,
  >						
  >   		 grade FLOAT,
  >						
  >   		 **PRIMARY KEY(stu_id,course_id)**
  >
  > );

**设置外键**

- CONSTRAINT 外键别名 FOREIGN KEY (属性1.1，属性1.2…属性1.n)

  REFERENCES  表名(属性2.1，属性2.2…属性2.n) 

  > CREATE TABLE example3(id INT PRIMARY KEY,
  >
  >  stu_id INT, 
  >
  > course_id INT,
  >
  > **CONSTRAINT c_fk FOREIGN KEY (stu id, course id)** 
  >
  > **REFERENCES example2(stu_id, course_id)** 
  >
  > );





## 3.2  查看表

先用use 数据库选择数据库，再用show tables查看该库的所有表

`use example；`

`show tables；`

- `desc 表名` 查看表的各属性信息
- `show create table 表名`查看表的完整创建语句,包括表的字段名、字段数据类型、完整约束条件信息，还可查看默认表的默认存储引擎和字符编码



## 3.3 修改表

- **修改表名**

`ALTER TABLE 旧表名 RENAME  [TO] 新表名；//TO为可选参数`

- **修改字段的数据类型**

`ALTER TABLE 表名 MODIFY 属性名 数据类型；`

`mysql> ALTER TABLE happy MODIFY stu_name VARCHAR(30);`

在修改数据类型时，原来有的非空约束会丢失，记得加上

- **修改字段的排列位置**

`ALTER TABLE 表名 MODIFY 属性名1 数据类型 FIRST | AFTER 属性名2；`

 字段修改到指定位置

` ALTER TABLE example MODIFY address VARCHAR(20)  AFTER phone;`

 <u>要加上数据类型，但不用加上约束性条件</u> 

- **修改字段名**

`ALTER TABLE 表名 CHANGE 旧属性名 新属性名 新数据类型；`

若不修改数据类型，则将新数据类型设置成原来一样，但是不能不写

- **增加字段**

`ALTER TABLE 表名 ADD属性名1 数据类型 [完整性约束条件] [FIRST] AFTER 属性名2；`

表的第一个位置增加字段

在example表中的第一个位置增加number字段并设置为主键

`ALTER TABLE example ADD number INT(20)  PRIMARY KEY  FIRST;`

- **删除字段**

`ALTER TABLE 表名 DROP 属性名；`

`ALTER TABLE example DROP  stu_id；`

- **更改表的存储引擎**

`ALTER TABLE 表名 ENGINE=存储引擎名；`

`ALTER TABLE example ENGINE=MyISAM；`

- **删除表的外键约束**

`ALTER TABLE 表名 DROP FOREIGN KEY 外键别名`

外键别名指创建时设置的外间的代号

`ALTER TABLE example3 DROP FOREIGN KEY c_fk;`

### 3.3.1 修改字符格式以支持中文

```mysql
mysql> insert into Student values('01', '赵磊', '1990-01-01', '男');
ERROR 1366 (HY000): Incorrect string value: '\xD5\xD4\xC0\xDA' for column 'Sname' at row 1
mysql> show variables like 'char%';
+--------------------------+---------------------------------------------------------+
| Variable_name            | Value                                                   |
+--------------------------+---------------------------------------------------------+
| character_set_client     | utf8                                                    |
| character_set_connection | utf8                                                    |
| character_set_database   | utf8                                                    |
| character_set_filesystem | binary                                                  |
| character_set_results    | utf8                                                    |
| character_set_server     | utf8                                                    |
| character_set_system     | utf8                                                    |
| character_sets_dir       | C:\Program Files\MySQL\MySQL Server 5.5\share\charsets\ |
+--------------------------+---------------------------------------------------------+
8 rows in set (0.00 sec)

mysql> set names 'gbk';
mysql> alter table Student character set gbk;
```

`ALTER TABLE  表名 character set  gbk `更改表的字符编码

`mysql> alter table Student character set gbk;`

## 3.4 删除表

- **清空表的内容**

`DELETE FROM 表名；`

- **删除没有关联的普通表**

`DROP TABLE 表名；`

- **删除被其他表关联的父表**

当一个表被另一个表关联，即有外键依赖于该表时，不能直接删除该表，要先删除外键约束

`ALTER TABLE 表名 DROP FOREIGN KEY 外键别名 ；`

用show create table查看外键是否被删除，若已删除，再删除父表

`DROP TABLE 表名；`



# 4.表数据操作

## 4.1 插入增加

- **为表的所有字段插入数据**

  > INSERT INTO 表名 VALUES(值1，值2，…值n);

​	   `insert into info values(15,'aaabd');//字符串类型必须加上引号`

- **为表的指定字段插入数据**

  > INSERT INTO 表名(属性a，属性c，…属性m)
  >
  > VALUES(值a，值c，…值m);

  列出所需字段即可，字段顺序随意，未设置的字段默认为NULL

- **同时插入多条记录**

  > INSERT INTO 表名 [(属性列表)]
  >
  > VALUES (取值列表1)，
  >
  > ​		(取值列表2)，…
  >
  > ​		(取值列表n)；

-  **将查询结果插入到表中**

  可将一个表的查询结果插入到另一个表中

  > INSERT INTO 表名 (属性列表1)
  >
  > ​		SELECT 属性列表2 FROM 表名2 WHERE 条件表达式

## 4.2 查询

> SELECT  [DISTINCT] 属性列表
>
> ​		FROM 表名和视图列表
>
> ​		[WHERE 条件表达式1]
>
> ​		[GROUP BY 属性名1 [HAVING 条件表达式2] ]
>
> ​		[ORDER BY 属性名2 [ASC | DESC] ]
>
> ​		[limit 前几行 [, 取几行] ]

- "属性列表"参数表示需要查询的字段名；

- "表名和视图列表"参数表示从此处指定的表或者视图中查询数据，表和视图可以有多个；

"条件表达式1"参数指定査询条件；

- "属性名1"参数指按该字段中的数据进行分组；

- “条件表达式2"参数表示满足该表达式的数据才能输出；

- "属性名2"参数指按该字段中的数据进行排序，排序方式由`ASC`和`DESC`两个参数指出；`ASC`参数表示按升序的顺序进行排序；`DESC`参数表示按降序的顺序进行排序，默认为`ASC`。

- 如果有`WHERE`子句，就按照"条件表达式1 "指定的条件进行査询；如果没有`WHERE`子句，就査询所有记录。

  |   查询条件   |          符号或关键字           |
  | :----------: | :-----------------------------: |
  |     比较     | =、<、<=、>、>=、!=、<>、!>、!< |
  |   比较范围   |  BETWEEN AND、NOT BETWEEN AND   |
  |   指定集合   |           IN、NOT IN            |
  |   匹配字符   |     LIKE、NOT LIKE、REGEXP      |
  |   是否为空   |      IS NULL、IS NOT NULL       |
  | 多个查询条件 |             AND、OR             |

  对于多个查询条件的`AND`和`OR`同时使用时，注意默认优先`AND`后`OR`，如`select * from table where a or b and c`时，等价于`a or (b and c)`，如果需要先使用`OR`则使用括号`(a or b ) and c`

- 如果有`GROUP BY`子句，就按照"属性名1"指定的字段进行分组；如果`GROUP BY`子句后带着HAVING关键字，那么只有满足"条件表达式2"中指定的条件的才能够输出。`GROUP BY`子句通常和`COUNT()`, `SUM()`等聚合函数一起使用。

- 如果有`ORDER BY`子句，就按照"属性名2"指定的字段进行排序。排序方式由`ASC`和`DESC`两个参数指出，默认的情况下是`ASC`。
- `limit 3 `表示取前三行， `limit 2,3`表示取第二行后的三行

### 4.2.1 检索多列

`mysql> select prod_id, prod_name, prod_price from products;`

### 4.2.2 检索全部

`mysql> select * from orders;`

### 4.2.3 检索不同的行

`mysql> select distinct vend_id from products;`

`DISTINCT`必须添加再列名前

### 4.2.4 限制行

`mysql> select prod_name from products limit 5;`取前5行

`mysql> select prod_name from products limit 2,5;` 取第二行后的前5行，如果不足5行则取至末行

### 4.2.5 检索结果排序

默认使用`order by` 后是升序`ASC`

`mysql> select prod_id, prod_price, prod_name from products order by prod_price desc;` 根据单列排序降序

`mysql> select prod_id, prod_price, prod_name from products order by prod_price desc, prod_name;` 根据多列排序，先全部行`prod_price`降序，后如果有相同的行则按照`prod_name`升序

### 4.2.6 使用`WHERE`条件过滤

|   查询条件   |          符号或关键字           |
| :----------: | :-----------------------------: |
|     比较     | =、<、<=、>、>=、!=、<>、!>、!< |
|   比较范围   |  BETWEEN AND、NOT BETWEEN AND   |
|   指定集合   |           IN、NOT IN            |
|   匹配字符   |     LIKE、NOT LIKE、REGEXP      |
|   是否为空   |      IS NULL、IS NOT NULL       |
| 多个查询条件 |             AND、OR             |

`mysql> select prod_name, prod_price from products where prod_price = 2.50;` 

`mysql> select prod_name, prod_price from products where prod_price between 5 and 10;` 

`mysql> select cust_id from customers where cust_email is null;`空值检查

`mysql> select prod_id, prod_price, prod_name from products where vend_id = 1003 and prod_price <= 10;`

`mysql> select vend_id, prod_name, prod_price from products where (vend_id = 1002 or vend_id = 1003) and prod_price >= 10;`

`mysql> select prod_name, prod_price from products where vend_id not in (1002,1003) and prod_price > 5 order by prod_name;`

`LIKE`需要完全一致才能匹配所在行，而`REGEXP`只要匹配部分既可

`LIKE`的匹配符`%`表示任何个（0个或1个或多个）字符出现任意次数，`_`只能匹配一个字符

**默认不区分大小写**

```mysql
mysql> select prod_name from products where prod_name like 'jet%';
+--------------+
| prod_name    |
+--------------+
| JetPack 1000 |
| JetPack 2000 |
+--------------+

mysql> select prod_name from products where prod_name like 'Jetpack__000';
+--------------+
| prod_name    |
+--------------+
| JetPack 1000 |
| JetPack 2000 |
+--------------+

// 使用regexp正则匹配只要存在'000'则可以匹配上
mysql> select prod_name from products where prod_name regexp '000';  
+--------------+
| prod_name    |
+--------------+
| JetPack 1000 |
| JetPack 2000 |
+--------------+
// 但是like必须要完全一致才能匹配上
mysql> select prod_name from products where prod_name like '000';
Empty set (0.00 sec)

```

### 4.2.7 计算字段

- 通过`concat(a,b,c,..)`函数拼接多个字段

  `mysql> select concat( vend_name, '(', vend_country, ')' ) from vendors order by vend_name;` 

- 在列表处对数据处理：算数计算`*、+、-、/`

  `mysql> select prod_id, quantity, item_price , quantity * item_price as expanded_price from orderitems where order_num = 20005;`

### 4.2.8 使用函数处理

一、数学函数
函数|含义
--|--
ABS(x) |  返回x的绝对值
BIN(x)   |返回x的二进制（OCT返回八进制，HEX返回十六进制）
EXP(x)  | 返回值e（自然对数的底）的x次方
FLOOR(x)|   返回小于x的最大整数值
GREATEST(x1,x2,...,xn)|返回集合中最大的值
LEAST(x1,x2,...,xn)          |  返回集合中最小的值
MOD(x,y)                             |     返回x/y的模（余数）
PI()|返回pi的值（圆周率）
RAND()|返回０到１内的随机值,可以通过提供一个参数(种子)
ROUND(x,y)|返回参数x的四舍五入的有y位小数的值
SIGN(x) |返回代表数字x的符号的值
SQRT(x) |返回一个数的平方根

二、聚合函数(常用于GROUP BY从句的SELECT查询中)
函数|含义
--|--
AVG(col)|返回指定列的平均值
COUNT(col)|返回指定列中非NULL值的个数
MIN(col)|返回指定列的最小值
MAX(col)|返回指定列的最大值
SUM(col)|返回指定列的所有值之和
GROUP_CONCAT(col) |返回由属于一组的列值连接组合而成的结果

三、字符串函数
函数|含义
--|--
ASCII(char)|返回字符的ASCII码值
CONCAT(s1,s2...,sn)|将s1,s2...,sn连接成字符串
CONCAT_WS(sep,s1,s2...,sn)|将s1,s2...,sn连接成字符串，并用sep字符间隔
INSERT(str,x,y,instr) |将字符串str从第x位置开始，y个字符长的子串替换为字符串instr，返回结果
FIND_IN_SET(str,list)|分析逗号分隔的list列表，如果发现str，返回str在list中的位置
LOWER(str) |返回将字符串str中所有字符改变为小写后的结果
LEFT(str,x)|返回字符串str中最左边的x个字符
LENGTH(s)|返回字符串str中的字符数
LTRIM(str) |从字符串str中切掉开头的空格
POSITION(substr,str)| 返回子串substr在字符串str中第一次出现的位置
REPEAT(str,srchstr,rplcstr)|返回字符串str重复x次的结果
REVERSE(str) |返回颠倒字符串str的结果
RIGHT(str,x) |返回字符串str中最右边的x个字符
RTRIM(str) |返回字符串str尾部的空格
STRCMP(s1,s2)|比较字符串s1和s2
TRIM(str)|去除字符串首部和尾部的所有空格
UPPER(str) |返回将字符串str中所有字符转变为大写后的结果

四、日期和时间函数
函数|含义
--|--
CURDATE()或CURRENT_DATE() |返回当前的日期
CURTIME()或CURRENT_TIME() |返回当前的时间
DATE_ADD(date,INTERVAL int keyword)|返回日期date加上间隔时间int的结果(int必须按照关键字进行格式化),如：SELECTDATE_ADD(CURRENT_DATE,INTERVAL 6 MONTH);
DATE_FORMAT(date,fmt) |   依照指定的fmt格式格式化日期date值
DATE_SUB(date,INTERVAL int keyword)|返回日期date加上间隔时间int的结果(int必须按照关键字进行格式化),如：SELECTDATE_SUB(CURRENT_DATE,INTERVAL 6 MONTH);
DAYOFWEEK(date)   |   返回date所代表的一星期中的第几天(1~7)
DAYOFMONTH(date)  |  返回date是一个月的第几天(1~31)
DAYOFYEAR(date)    |  返回date是一年的第几天(1~366)
DAYNAME(date)     | 返回date的星期名，如：SELECT DAYNAME(CURRENT_DATE);
FROM_UNIXTIME(ts,fmt)  |  根据指定的fmt格式，格式化UNIX时间戳ts
HOUR(time)    |  返回time的小时值(0~23)
MINUTE(time)  |    返回time的分钟值(0~59)
MONTH(date)    |  返回date的月份值(1~12)
MONTHNAME(date)    |  返回date的月份名，如：SELECT MONTHNAME(CURRENT_DATE);
NOW()     |   返回当前的日期和时间
QUARTER(date)  |    返回date在一年中的季度(1~4)，如SELECT QUARTER(CURRENT_DATE);
WEEK(date)  |    返回日期date为一年中第几周(0~53)
YEAR(date)   |   返回日期date的年份(1000~9999)
获取当前系统时间：|SELECT FROM_UNIXTIME(UNIX_TIMESTAMP());
获取当前系统时间：|SELECT EXTRACT(YEAR_MONTH FROM CURRENT_DATE);
获取当前系统时间：|SELECT EXTRACT(DAY_SECOND FROM CURRENT_DATE);
获取当前系统时间：|SELECT EXTRACT(HOUR_MINUTE FROM CURRENT_DATE);
返回两个日期值之间的差值(月数)：|SELECT PERIOD_DIFF(200302,199802);

`mysql> select cust_id,order_num from orders where date(order_date) between '2005-09-01' and '2005-09-30';`

`mysql> select cust_id, order_num from orders where year(order_date) = 2005 and month(order_date) = 9;`

`mysql> select avg(prod_price) as avg_price from products where vend_id = 1003;`

使用 `AVG() `函数返回特定供应商提供的产品的平均价格。
它与上面的 `SELECT `语句相同，但使用了` DISTINCT `参数，因此平均值只考虑各个不同的价格：

`mysql> select avg(DISTINCT prod_price) as avg_price from products where vend_id = 1003;`

`mysql> select count(*) as num_list from customers;`

通过`AS`设置别名，设置的别名后面也可以使用，后面详解

### 4.2.9 `GROUP BY` 分组

`GROUP BY 属性名  [HAVING 条件表达式]  [WITH ROLLUP]`

**`GROUP BY `子句必须出现在` WHERE` 子句之后， `ORDER BY` 子句之前。**

`GROUP BY`后接`HAVING`过滤分组
`WHERE` 过滤行，而 `HAVING` 过滤分组。`HAVING `支持所有` WHERE` 操作符 

`mysql> select cust_id, count(*) as num_orders from orders group by cust_id having count(*) >= 2;`

`mysql> select vend_id, count(*) as num_prods from products where prod_price >= 10 group by vend_id having num_prods >= 2;`

`GROUP BY` 后使用 `ORDER BY`，对分组后的组排序

`mysql> select order_num, sum(quantity * item_price) as ordertotal from orderitems  group by order_num having ordertotal >= 50 order by ordertotal limit 1,2;`

`WITH ROLLUP`关键字将会在所有记录的最后加上一条记录，该记录是上面所有记录的总和。

### 4.2.10 子查询

先查表一获得一个结果，然后取表二根据表一获得的结果查询，分了两条语句

```mysql
select order_num from orderitems where prod_id = 'TNT2';  // 得出结果(20005, 20007)
select cust_id from orders where order_num in (20005, 20007);
```

可以合并两条语句为一条，通过子查询，**将查询的结果作为条件**

```mysql
mysql> select cust_name, cust_contact from customers where cust_id in
    -> ( select cust_id from orders where order_num in
    -> ( select order_num from orderitems where prod_id = 'TNT2')); 
```

**将查询的结果作为属性列表**

```mysql
select cust_name, cust_state, 
(select count(*) from orders where orders.cust_id = customers.cust_id) as orders
from customers order by cust_name;
```

### 4.2.11 内联结

可以联结两个表

`mysql> select vend_name, prod_name, prod_price from vendors, products where vendors.vend_id = products.vend_id order by vend_name, prod_name;`

等价于通过`select * from 表一 inner join 表二 on 表一.列名1 = 表二.列名1`

`mysql> select vend_name, prod_name, prod_price from vendors inner join products on vendors.vend_id = products.vend_id;`

联结多个表

```mysql
mysql> select prod_name, vend_name, prod_price, quantity from orderitems, products, vendors
    ->  where products.vend_id = vendors.vend_id and
    -> orderitems.prod_id = products.prod_id;
```



笛卡尔积`select * from vendors, products;`表一行数 * 表二行数

### 4.2.12 表别名、自联结

`AS`设置表别名，可以在属性列表中设置，也可以在表位置设置

**通过属性列表设置**，后面也可以使用该表别名

`select concat( rtrim(vend_name), '(', rtrim(vend_country), ')' ) as vend_title from vendors order by vend_title;`

**给不同表起不同的别名**，方便后面语句书写

`mysql> select cust_name, cust_contact from customers as c, orders as o, orderitems as i where o.order_num = i.order_num and c.cust_id = o.cust_id and prod_id = 'TNT2';`

**给同一张表起不同的别名**，实现**自联结**

> 假如你发现某物品（其ID为 DTNTR ）存在问题，因此想知道生产该物
> 品的供应商生产的其他物品是否也存在这些问题。此查询要求首先找到
> 生产ID为 DTNTR 的物品的供应商，然后找出这个供应商生产的其他物品。
> 下面是解决此问题的一种方法：

```mysql
select prod_id, prod_name from products where vend_id = (select vend_id from products where prod_id = 'DTNTR');

select p1.prod_id, p1.prod_name from products as p1, products as p2 
-> where p1.vend_id = p2.vend_id and p2.prod_id = 'DTNTR';
```

### 4.2.13 外联结

`select * from 表一 LEFT OUTER JOIN 表二 ON 表一.属性1 = 表二.属性1;`

`select * from 表一 RIGHT OUTER JOIN 表二 ON 表一.属性1 = 表二.属性1;`

### 4.2.14 组合查询

> SELECT a,b,c FROM 表一...
>
> UNION [ALL]
>
> SELECT a,b,c FROM 表二...
>
> UNION [ALL]
>
> SELECT a,b,c FROM 表三...
>
> 将第一条查询的结果和第二条查询的结果上下拼接，属性列表需要相同
>
> 默认去除重复的行，通过加上 ALL 保留重复的行
>
> UNION 必须由两条或两条以上的 SELECT 语句组成，语句之间用关
> 键字 UNION 分隔（因此，如果组合4条 SELECT 语句，将要使用3个
> UNION 关键字）。



```mysql
mysql> select vend_id, prod_id from products where prod_price < 5
    -> union
    -> select vend_id, prod_id from products where vend_id in (1001, 1002);
```





## 4.3 删除

>  DELETE FROM 表名 [WHERE  条件表达式]；
>
> 如果不加条件表达式则删除整个表格数据，但是表仍然存在

如果要清空整张表推荐使用`TRUNCATE TABLE`，创建新的的空表，并删除原表，不会一行一行删除数据

## 4.4 更改

> UPDATE 表名SET 
>
> 属性名1=取值1，
>
> 属性名2=取值2，
>
> …
>
> 属性名n=取值n
>
> WHERE 条件表达式；



