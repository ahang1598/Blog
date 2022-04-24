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

练习方式：通过[牛客网](https://www.nowcoder.com/ta/sql)或[力扣](https://leetcode-cn.com/problemset/database/)来刷题



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
>
> ​		[ OFFSET 偏移N行]

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

- 取第三行`limit 1 offset 2`

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
ROUND(x,y)|返回参数x的四舍五入的有y位小数的值 [题目](https://www.nowcoder.com/practice/f41b94b4efce4b76b27dd36433abe398?tpId=82&tags=&title=&difficulty=0&judgeStatus=0&rp=1&gioEnter=menu)
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

### 4.2.15 函数排名

1、RANK()

  在计算排序时，若存在相同位次，会跳过之后的位次。

  例如，有3条排在第1位时，排序为：1，1，1，4······

2、DENSE_RANK()

  这就是题目中所用到的函数，在计算排序时，若存在相同位次，不会跳过之后的位次。

  例如，有3条排在第1位时，排序为：1，1，1，2······

3、ROW_NUMBER()

  这个函数赋予唯一的连续位次。

  例如，有3条排在第1位时，排序为：1，2，3，4······



窗口函数用法：

<窗口函数> OVER ( [PARTITION BY <列清单> ]

​                ORDER BY <排序用列清单> ）

例题[sql23](https://www.nowcoder.com/practice/b9068bfe5df74276bd015b9729eec4bf?tpId=82&tags=&title=&difficulty=0&judgeStatus=0&rp=1)

```mysql
select emp_no, salary,
       dense_rank() over (order by salary desc) as rank
from salaries
where to_date='9999-01-01'
order by rank asc,emp_no asc;
```

通过非函数方法排名

```mysql
-- rank排名：查询表中大于自己薪水的员工的数量（考虑并列：去重）
SELECT 
  s1.emp_no,
  s1.salary,
  (SELECT 
    COUNT(DISTINCT s2.salary) 
  FROM
    salaries s2 
  WHERE s2.to_date = '9999-01-01' 
    AND s2.salary >= s1.salary) AS `rank`  -- 去重：计算并列排名
FROM
  salaries s1 
WHERE s1.to_date = '9999-01-01' 
ORDER BY s1.salary DESC,
  s1.emp_no ;
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



# 5.SQL优化

## 5.1执行计划

MySQL 使用 `explain + sql 语句`查看 执行计划，该执行计划不一定完全正确但是可以参考。

```
EXPLAIN SELECT * FROM user WHERE nid = 3;
```

![img](https://img2018.cnblogs.com/blog/1334255/201812/1334255-20181218161456575-1277410542.png)

| select_type  |      说明      |
| :----------: | :------------: |
|    SIMPLE    |    简单查询    |
|   PRIMARY    |   最外层查询   |
|   SUBQUERY   |  映射为子查询  |
|   DERIVED    |     子查询     |
|    UNION     |      联合      |
| UNION RESULT | 使用联合的结果 |

------

**table** : 正在访问的表名

------

|                             type                             |                             说明                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                             ALL                              |                         全数据表扫描                         |
|                            index                             |                         全索引表扫描                         |
|                            RANGE                             |                     对索引列进行范围查找                     |
|                         INDEX_MERGE                          |                合并索引，使用多个单列索引搜索                |
|                             REF                              |                   根据索引查找一个或多个值                   |
|                            EQ_REF                            |             搜索时使用primary key 或 unique类型              |
|                            CONST                             | 常量，表最多有一个匹配行,因为仅有一行,在这行的列值可被优化器剩余部分认为是常数,const表很快,因为它们只读取一次。 |
|                            SYSTEM                            |   系统，表仅有一行(=系统表)。这是const联接类型的一个特例。   |
| 性能：`all` < `index` < `range` < `index_merge` < `ref_or_null` < `ref` < `eq_ref` < `system/const` |                                                              |
|             性能在 range 之下基本都可以进行调优              |                                                              |

------

**possible_keys** : 可能使用的索引

------

**key** : 真实使用的

------

**key_len** : MySQL中使用索引字节长度

------

**rows** : mysql 预估为了找到所需的行而要读取的行数

------

|                    extra                    |                             说明                             |
| :-----------------------------------------: | :----------------------------------------------------------: |
|                 Using index                 |         此值表示mysql将使用覆盖索引，以避免访问表。          |
|                 Using where                 | mysql 将在存储引擎检索行后再进行过滤，许多where条件里涉及索引中的列，当(并且如果)它读取索引时，就能被存储引擎检验，因此不是所有带where子句的查询都会显示“Using where”。有时“Using where”的出现就是一个暗示：查询可受益于不同的索引。 |
|               Using temporary               |             mysql 对查询结果排序时会使用临时表。             |
|               Using filesort                | mysql会对结果使用一个外部索引排序，而不是按索引次序从表里读取行。mysql有两种文件排序算法，这两种排序方式都可以在内存或者磁盘上完成，explain不会告诉你mysql将使用哪一种文件排序，也不会告诉你排序会在内存里还是磁盘上完成。 |
| Range checked for each record(index map: N) | 没有好用的索引，新的索引将在联接的每一行上重新估算，N是显示在possible_keys列中索引的位图，并且是冗余的 |

**limit**

limit 匹配后就不会继续进行扫描

```
mysql> SELECT * FROM user WHERE email = 'klvchen123@126.com' LIMIT 1;
+-----+------------+--------------------+-------+
| nid | name       | email              | extra |
+-----+------------+--------------------+-------+
| 123 | klvchen123 | klvchen123@126.com | 123   |
+-----+------------+--------------------+-------+
1 row in set (0.01 sec)
```



详细参看[博客](https://www.cnblogs.com/ggjucheng/archive/2012/11/11/2765237.html)或[备份](pages/mysql_explain.html)



## 5.2正确使用索引

1. 使用 like 语句时，%在右边才会使用索引。
   `没用使用索引`
   ![img](https://img2018.cnblogs.com/blog/1334255/201812/1334255-20181219104551833-174899627.png)
   `使用索引`
   ![img](https://img2018.cnblogs.com/blog/1334255/201812/1334255-20181219114640747-1725821256.png)
2. or 条件中有未建立索引的列才,索引失效
   `没用使用索引`
   ![img](https://img2018.cnblogs.com/blog/1334255/201812/1334255-20181219115507465-1096074692.png)
   `使用索引`
   ![img](https://img2018.cnblogs.com/blog/1334255/201812/1334255-20181219115236816-1647665968.png)
3. 条件的类型不一致
   `没用使用索引`
   ![img](https://img2018.cnblogs.com/blog/1334255/201812/1334255-20181219134139518-222220297.png)
   `使用索引`
   ![img](https://img2018.cnblogs.com/blog/1334255/201812/1334255-20181219134204585-264948476.png)
4. != 号
   `没用使用索引`
   ![img](https://img2018.cnblogs.com/blog/1334255/201812/1334255-20181219134317541-1609586249.png)
   例外：如果是主键，则会走索引
5. \> 号
   `没用使用索引`
   ![img](https://img2018.cnblogs.com/blog/1334255/201812/1334255-20181219134433905-1062967837.png)
   例外：如果是主键或索引是整数类型，则会走索引
6. order by
   `没用使用索引`
   ![img](https://img2018.cnblogs.com/blog/1334255/201812/1334255-20181219134722295-1809068294.png)
   例外：如果 order by 是主键或索引是整数类型，则会走索引
7. 组合索引
   遵循最左前缀

```
# 若 name 和 email 组成组合索引
create index ix_name_email on user(name,email);

name and email -- 使用索引
email and name -- 不使用索引
name                  -- 使用索引
email                  -- 不使用索引
```

`没用使用索引`
![img](https://img2018.cnblogs.com/blog/1334255/201812/1334255-20181219135548420-1350585399.png)
![img](https://img2018.cnblogs.com/blog/1334255/201812/1334255-20181219135612019-804744999.png)





# 6. 



![](img\MySQL\167f4c.awebp)



**客户端连接服务端**

通过`TCP/IP`加`端口`连接 `mysql -h127.0.0.1 -uroot -P3306 -p`

 也可以通过命名管道或共享内存 、 Unix域套接字



**查询缓存**

如果**两个查询请求在任何字符上的不同**（例如：空格、注释、大小写），都会导致**缓存不会命中**。

另外，如果查询请求中包含某些系统函数、用户自定义变量和函数、一些系统表，如mysql 、information_schema、 performance_schema 数据库中的表，那这个**请求就不会被缓存**。

以某些系统函数举例，可能同样的函数的两次调用会产生不一样的结果，比如函数 NOW ，每次调用都会产生最新的当前时间，如果在一个查询请求中调用了这个函数，那即使查询请求的文本信息都一样，那不同时间的两次查询也应该得到不同的结果，如果在第一次查询时就缓存了，那第二次查询的时候直接使用第一次查询的结果就是错误的！
不过既然是缓存，那就有它缓存失效的时候。MySQL的缓存系统会监测涉及到的每张表，**只要该表的结构或者数据被修改，如对该表使用了 INSERT** 、 UPDATE 、 DELETE 、 TRUNCATE TABLE 、 ALTER TABLE 、 DROP TABLE 或
DROP DATABASE 语句，那使用**该表的所有高速缓存查询都将变为无效并从高速缓存中删除**！

虽然查询缓存有时可以提升系统性能，但也不得不因维护这块缓存而造成一些开销，比如每次都要去查
询缓存中检索，查询请求处理完需要更新查询缓存，维护该查询缓存对应的内存区域。

**从MySQL 5.7.20开始，不推荐使用查询缓存，并在MySQL 8.0中删除**。



# 7. 修改字符集

修改**服务器**默认字符集为`utf8`

进入`C:\ProgramData\MySQL\MySQL Server 5.7`修改`my.ini`

`character-set-server=utf8`

重启服务器后查看

`show VARIABLES like 'CHARACTER_set_server';`

比较规则`show VARIABLES like 'COLLATION_server';`



修改**数据库**默认字符集utf8

进入某数据库后

`ALTER DATABASE CHARACTER SET utf8;`

查看` SHOW VARIABLES LIKE 'character_set_database';`

CREATE DATABASE 数据库名
[[DEFAULT] CHARACTER SET 字符集名称]
[[DEFAULT] COLLATE 比较规则名称];

ALTER DATABASE 数据库名
[[DEFAULT] CHARACTER SET 字符集名称]
[[DEFAULT] COLLATE 比较规则名称];



表级别

修改`ALTER table tb_user CHARACTER SET 'utf8'; `

查看`show CREATE table tb_user`

CREATE TABLE 表名 (列的信息)
[[DEFAULT] CHARACTER SET 字符集名称]
[COLLATE 比较规则名称]]
ALTER TABLE 表名
[[DEFAULT] CHARACTER SET 字符集名称]
[COLLATE 比较规则名称]



![](C:\Users\Ahang\AppData\Roaming\Typora\typora-user-images\image-20220422171939024.png)

服务器认为客户端发送过来的请求是用 character_set_client 编码的。
假设你的**客户端采用的字符集和 character_set_client 不一样的话，这就会出现意想不到的**情况。比如我的客户端使用的是 utf8 字符集，如果把系统变量 character_set_client 的值设置为 ascii 的话，服务器可能无法理解我们发送的请求，更别谈处理这个请求了。

服务器将把得到的结果集使用 character_set_results 编码后发送给客户端。
假设你的**客户端采用的字符集和 character_set_results 不一样的话，这就可能会出现客户端无法解码**结果集的情况，结果就是在你的屏幕上出现乱码。比如我的客户端使用的是 utf8 字符集，如果把系统变character_set_results 的值设置为 ascii 的话，可能会产生乱码。

**character_set_connection** 只是服务器在将请求的字节串从 character_set_client 转换为character_set_connection 时使用，它是什么其实没多重要，但是一定要注意，**该字符集包含的字符范围一定涵盖请求中的字符，要不然会导致有的字符无法使用 character_set_connection 代表的字符集进行编码**。比如你把 character_set_client 设置为 utf8 ，把 character_set_connection 设置成 ascii ，那么此时你如果从客户端发送一个汉字到服务器，那么服务器无法使用 ascii 字符集来编码这个汉字，就会向用户发出一个警告。



我们通常都把 character_set_client 、character_set_connection、character_set_results 这三个系统变量设置成和客户端使用的字符集一致的情况，
`SET NAMES 字符集名;`

如`SET NAMES utf8;`

这一条语句产生的效果和我们执行这3条的效果是一样的：
SET character_set_client = 字符集名;
SET character_set_connection = 字符集名;
SET character_set_results = 字符集名;



**修改比较规则**

比较规则 是针对某个字符集中的字符比较大小的一种规则

默认比较规则不比较大小写，可以将`utf8_general_ci`修改为`utf_bin`

```mysql
 show VARIABLES LIKE 'COLLATION_database';
 
 CREATE table t ( col VARCHAR(20) );
 
 INSERT INTO t(col) VALUES ('a'), ('b'), ('B'), ('A'), ('我');
 
 select * from t order by col asc;
 
 ALTER TABLE t MODIFY col VARCHAR(20) COLLATE utf8_bin;
```





# 8.InnoDB



**页**

将数据划分为若干个页，**以页作为磁盘和内存之间交互的基本单位**，**InnoDB中页的大小一般为 16 KB**。也就是在一般情况下，一次最少从磁盘中读取16KB的内容到内存中，一次最少把内存中的16KB内容刷新到磁盘中。

**行格式**

记录在磁盘上的存放方式也被称为 行格式 或者 记录格式 

Compact 、 Redundant 、Dynamic 和 Compressed 行格式

**指定行格式的语法**

CREATE TABLE 表名 (列的信息) ROW_FORMAT=行格式名称
ALTER TABLE 表名 ROW_FORMAT=行格式名称



## Compact格式

![](img\MySQL\image-20220423093408148.png)

**变长字段长度列表**

把所有变长字段的真实数据占用的**字节长度**都存放在记录的开头部位，各变长字段数据占用的字节数按照列的顺序逆序存放

变长字段长度列表中**只存储值为 非NULL 的列内容占用的长度**，值为 NULL 的列的长度是不储存的 

**NULL值列表**

按照列位置若为NULL则置二进制bit位为1，否则置0，倒序排序，比如`00000101`则表示第一列和第三列数值为NULL

**记录头**

![image-20220423095056045](img\MySQL\image-20220423095056045.png)

名称 |大小（单位：bit） |描述
--|--|--
预留位1| 1| 没有使用
预留位2| 1| 没有使用
delete_mask |1| 标记该记录是否被删除
min_rec_mask  |1 | B+树的每层非叶子节点中的最小记录都会添加该标记
n_owned  |4 | 表示当前记录拥有的记录数
heap_no  |13  |表示当前记录在记录堆的位置信息
record_type  |3  |表示当前记录的类型， 0 表示普通记录， 1 表示B+树非叶子节点记录， 2 表示最小记录， 3表示最大记录
next_record  |16  |表示下一条记录的相对位置



**隐藏列 记录的真实数据**

![image-20220423095504859](img\MySQL\image-20220423095504859.png)



一条完整的记录

![](img\MySQL\image-20220423095004659.png)



## **行溢出数据**

### VARCHAR(M)最多能存储的数据

 VARCHAR(M) 类型的列最多可以占用 65535 个字节

1.  ascii 字符集的话，一个字符就代表一个字节，
   如果该 VARCHAR 类型的列没有 NOT NULL 属性，那最多只能存储 65532 个字节的数据，因为真实数据的长度可能占用2个字节， NULL 值标识需要占用1个字节

   如果 VARCHAR 类型的列有 NOT NULL 属性，那最多只能存储 65533 个字节的数据，因为真实数据的长度可能占用2个字节，不需要 NULL 值标识

2.  gbk 字符集表示一个字符最多需要 2 个字节，那在该字符集下， M 的最大取值就是 32766 （也就是：65532/2），也就是说最多能存储 32766 个字符；

3. utf8 字符集表示一个字符最多需要 3 个字节，那在该字符集下， M 的最大取值就是 21844 ，就是说最多能存
   储 21844 （也就是：65532/3）个字符

### 行溢出分页

一个页的大小一般是 16KB ，也就是 16384 字节，而一个 VARCHAR(M) 类型的列就最多可以存储 65532 个字节，这样就可能造成一个页存放不了一条记录的尴尬情况

在 **Compact 和 Reduntant** 行格式中，在 记录的真实数据 处只会存储该列的一部分数据，把剩余的数据分散存储在几个其他的页中，然后 **记录的真实数据 处用20个字节存储指向这些页的地址**

在本记录的真实数据处只会存储该列的前 768 个字节的数据和一个指向其他页的地址，然后把剩下的数据存放到其他页中，这个过程也叫做 **行溢出** ，存储超出 768 字节的那些页面也被称为 **溢出页** 



不只是 **VARCHAR(M)** 类型的列，其他的 **TEXT、BLOB** 类型的列在存储数据非常多的时候也会发生 **行溢出** 。



MySQL 中规定一个页中至少存放两行记录



## Dynamic和Compressed行格式

两格式和Compact基本一样，只是在处理溢出页不同

对于会出现溢出时，会将**所有的数据全都放一个单独的页面**，然后指向该页面

 Compressed 行格式会采用压缩算法对页面进行压缩，以节省空间。



## 数据页结构

一个 InnoDB 数据页的存储空间

![image-20220423105349509](img\MySQL\image-20220423105349509.png)



### 记录

一个**页**内有多个**记录**

**记录**按照主键从小到大的顺序形成了一个**单链表**，单链表主要通过next_record中偏移位来计算

当某一记录被删后会将该记录中`delete_mask`置`1`，等后面插入记录时会替换，或者统一回收处理，而不会立刻删除

默认一页存在两个默认记录，最小和最大记录

对于最小记录所在的分组只能有 1 条记录，最大记录所在的分组拥有的记录条数只能在 1~8 条之间，剩下的分组中记录的条数范围只能在是 4~8 条之间

![image-20220423194920689](img\MySQL\image-20220423194920689.png)



### Page Directory页目录

同一页内部分组

记录分组是为了方便查找，分组后将每组的**最大节点**位置放入**页目录**中，然后通过**二分法查找**

![image-20220423200100074](img\MySQL\image-20220423200100074.png)



### File Header

包含 校验和

上下页号：每页之间形成双向链表

![image-20220423202159445](img\MySQL\image-20220423202159445.png)



### File Trailer

校验和：这个部分是和 **File Header** 中的校验和相对应的

每当一个页面在内存中修改了，在同步之前就要把它的校验和算出来，因为 File Header 在页面的前边，所以校验和会被首先同步到磁盘，当完全写完时，校验和也会被写到页的尾部，如果完全同步成功，则页的首部和尾部的校验和应该是一致的。如果写了一半儿断电了，那么在 File Header 中的校验和就代表着已经修改过的页，而在 File Trialer 中的校验和代表着原先的页，二者不同则意味着同步中间出了错。



## 索引



### 没有索引时查找

`select * from user where id = 10;`

同一页中查找：

如果id为主键时，直接根据页目录二分定位记录所在组，然后遍历该组中记录

如果id不是主键时，只能从第一组开始顺序查找



多数据页查找：

顺序查找每页，然后在每页内查找。











