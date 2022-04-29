# 1. 使用MySQL

## 1.1 登录数据库
`mysq-h 127.0.0.1 -u root -p****`
- -h指连接的主机名，因为连接的本机，所以为localhost；-
- -u表示用户名，此处为root；
- -p指用户密码，可以在-p后面紧接着输入密码。

```mysql
mysql> exit # 退出 使用 “quit;” 或 “\q;” 一样的效果
mysql> status;  # 显示当前mysql的version的各种信息
mysql> select version(); # 显示当前mysql的version信息
mysql> show globavariables like 'port'; # 查看MySQL端口号
```
## 1.2 cmd管理员模式开启关闭MySQL服务
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



Linux下忘记密码修改：

1. `vi /etc/my.cnf`，在[mysqld]下面添加一句`skip-grant-tables`

2. `service mysqld restart`重启服务

3. 直接通过`mysql`命令进入
   ```mysql
   use mysql;
   update user set authentication_string=password('root') where User='root';
   flush privileges;
   exit
   ```

4. `vi /etc/my.cnf`，在[mysqld]下面删除一句`skip-grant-tables`

5. `systemctl restart mysqld`重启服务

6. 正常进入`mysql -uroot -p`



# 2. 数据库基本操作



- 创建数据库 `CREATE DATABASE 数据库名;`
- 删除数据库 `DROP DATABASE 数据库名;`
- 查看已存在数据库 `SHOW DATABASES;`
- 选择数据库 `USE 数据库名;`
- 导入SQL脚本`mysql> source D:\populate.sql` 



## 2.1 数据库设计范式

第一范式1NF：一个关系模式R的所有属性都是不可分割的基本数据项。

第二范式：在1NF的基础上，非码属性必须完全依赖于候选码（在1NF基础上消除非主属性对主码的部分函数依赖）
					：一个表必须有主键,即每行数据都能被唯一的区分

第三范式：在2NF基础上，任何非主属性不依赖于其它非主属性（在2NF基础上消除传递依赖）
					：一个表中不能包含其他相关表中非关键字段的信息,即数据表不能有冗余字段

![image-20220428091348985](img\MySQL\image-20220428091348985.png)

## 2.2 反范式

反范式化是指为了查询效率的考虑把原本符合第三范式的表“适当”的增加冗余，以达到优化查询效率的目的，反范式化是一种以空间来换取时间的操作。

将多个分隔的符合第三范式的表通过主键连接一张表，减少后面操作时对表的连接成本，而且可以建立更多符合搜索条件的索引。但是增加了空间。

![image-20220428091357390](img\MySQL\image-20220428091357390.png)





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
  `id`             int(100) unsigned NOT NULAUTO_INCREMENT primary key,
  `password`       varchar(32)       NOT NULDEFAULT '' COMMENT '用户密码',
  `reset_password` tinyint(32)       NOT NULDEFAULT 0 COMMENT '用户类型：0－不需要重置密码；1-需要重置密码',
  `mobile`         varchar(20)       NOT NULDEFAULT '' COMMENT '手机',
  `create_at`      timestamp(6)      NOT NULDEFAULT CURRENT_TIMESTAMP(6),
  `update_at`      timestamp(6)      NOT NULDEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
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
  ```mysql
  CREATE TABLE example2(
      stu_id INT,
  	 course_id INT,
  	 grade FLOAT,
  	 PRIMARY KEY(stu_id,course_id)
  );    
  ```

**设置外键**

- CONSTRAINT 外键别名 FOREIGN KEY (属性1.1，属性1.2…属性1.n)

  REFERENCES  表名(属性2.1，属性2.2…属性2.n) 

  ```mysql
  CREATE TABLE example3(id INT PRIMARY KEY,
  	stu_id INT, 
  	course_id INT,
  	CONSTRAINT c_fk FOREIGN KEY (stu id, course id) 
  	REFERENCES example2(stu_id, course_id) 
  );
  ```
  
  

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

同时添加多列

`alter table example_table add a int, add b int;`

- **删除字段**

`ALTER TABLE 表名 DROP 属性名；`

`ALTER TABLE example DROP  stu_id；`

同时删除多列

`alter table example drop column1, drop column2;`

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
| character_sets_dir       | C:\Program Files\MySQL\MySQServer 5.5\share\charsets\ |
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

`mysql> select cust_id from customers where cust_emaiis null;`空值检查

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
DATE_ADD(date,INTERVAint keyword)|返回日期date加上间隔时间int的结果(int必须按照关键字进行格式化),如：SELECTDATE_ADD(CURRENT_DATE,INTERVA6 MONTH);
DATE_FORMAT(date,fmt) |   依照指定的fmt格式格式化日期date值
DATE_SUB(date,INTERVAint keyword)|返回日期date加上间隔时间int的结果(int必须按照关键字进行格式化),如：SELECTDATE_SUB(CURRENT_DATE,INTERVA6 MONTH);
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

`mysql> select order_num, sum(quantity * item_price) as ordertotafrom orderitems  group by order_num having ordertota>= 50 order by ordertotalimit 1,2;`

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

![](img\MySQL\sql-join.png)

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
> 默认去除重复的行，通过加上 AL保留重复的行
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



### 4.2.16 @与@@

`@自定义变量`定义一个本地变量

```mysql
mysql> set @a = 1;
Query OK, 0 rows affected (0.00 sec)

mysql> selete @a; # 错误，需要结合表使用
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'selete @a' at line 1

mysql> select @a as id , name from t1;
+------+------+
| id   | name |
+------+------+
|    1 | gaga |
|    1 | haha |
+------+------+

# 实现以5递增
mysql> select (@i := @i + 5) as id, name from t1, (select @i := 100) as init;
+------+------+
| id   | name |
+------+------+
|  105 | gaga |
|  110 | haha |
+------+------+
```



`@@系统变量`使用系统变量

```mysql
mysql> show variables like '%character%';
+--------------------------+-----------------------------------+
| Variable_name            | Value                             |
+--------------------------+-----------------------------------+
| character_set_client     | utf8                              |
| character_set_connection | utf8                              |
| character_set_database   | utf8mb4                           |
| character_set_filesystem | binary                            |
| character_set_results    | utf8                              |
| character_set_server     | utf8mb4                           |
| character_set_system     | utf8                              |
| character_sets_dir       | /www/server/mysql/share/charsets/ |
+--------------------------+-----------------------------------+
8 rows in set (0.00 sec)

mysql> set @@character_set_database = gbk;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> select @@character_set_database;
+--------------------------+
| @@character_set_database |
+--------------------------+
| gbk                      |
+--------------------------+
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



## 4.5 触发器

触发器是MySQL响应以下任意语句而自动执行的一条MySQL语句

（或位于 BEGIN 和 END 语句之间的一组语句）：

- DELETE ；
- INSERT ；
- UPDATE 。

仅支持表 只有表才支持触发器，视图不支持（临时表也不支持）。



## 4.6 创建删除索引

### 4.6.1 普通、主键、唯一索引区别

普通索引基本没有约束

唯一索引：要求插入的值不能重复

主键索引：1. 在唯一索引的基础上，不允许有空值，且只允许有一个存在。2. 唯一的聚集索引。

主键就是一种特殊唯一性索引。

### 4.6.2 创建

key 与 index都可以表示索引，但是

主键索引（必须指定为“PRIMARY KEY”，没有PRIMARY Index）、

唯一索引（unique index，一般写成unique key）、

普通索引(index，只有这一种才是纯粹的index)



```mysql
CREATE TABLE 表名(属性名 数据类型 [完整性约束条件]，
 	  	 属性名 数据类型 [完整性约束条件]，
 		 属性名 数据类型 
		[UNIQUE | FULLTEXT ] INDEX | KEY [别名] （属性名1 [(长度)]  [ASC|DESC]）
);
```



- UNIQUE是可选参数，表示索引为唯一性索索引；

- FULLTEXT是可选参数，表示索引为全文索引；

- INDEX和KEY参数用来指定字段为索引的，两者选择其中之一就可以了，作用一样；

- "别名"是可选参数，用来给创建的索引取的新名称；

- "属性1“参数指定索引对应的字段的名称，该字段必须为前面定义好的字段；

- "长度"是可选参数，其指索引的长度，必须是字符串类型才可以使用；

- "ASC"和"DESC”都是可选参数，"ASC"参数表示升序排列，"DESC"参数表示降序排列。

**创建普通索引**

创建一个普通索索时，不需要加任何UNIQUE、 FULLTEXT或者SPATIRL参数。

创建一个表名为index1的表，在表中的id字段上建立索引。

```mysql
mysql> CREATE TABLE index1(
  			id INT,
  			name VARCHAR(20),
  			sex BOOLEAN,
  			INDEX(id)
);
```



**创建唯一性索引**

需要使用`UNIQUE`参数来约束

创建一个表名为index2的表，在表中的id字段上建立名为index2_id的唯一性索引，且以升序形式排列

```mysql
mysql> CREATE TABLE index2(
  		id INT UNIQUE,
  		name VARCHAR(20),
  		UNIQUE INDEX index2_id(id ASC)
);
```



**创建全文索引**

全文索引只能创建在CHAR、VARCHAR或TEXT类型的字段上，而且**只有MyISAM存储引擎支持**全文索引

//创建一个表名为index3的表，在表中的info字段上建立名为index3_info的全文索引

mysql> CREATE TABLE index3(

  -> id INT,

  -> info VARCHAR(20),

  -> FULLTEXT INDEX index3_info(info)

  -> )ENGINE=MyISAM;

**创建单列索引**

单列索引是在表的单个字段上创建索引

//创建index4表，在subject字段上建立名为index4_st的单列索引

```mysql
mysql> CREATE TABLE index4(
  -> id INT,
  -> subject VARCHAR(30),
  -> INDEX index4_st(subject(10))
  -> );
```

subject字段长度为20，而**index4_st索引长度只有10**，是为了提高查询速度，对于字符型数据，不用查询全部信息，只查询其前面的若干字符信息。

 **创建多列索引**

多列索引是在表的多个字段上创建索引

```mysql
mysql> CREATE TABLE index5(
  -> id INT,
  -> name CHAR(20),
  -> sex CHAR(4),
  -> INDEX index5_ns(name,sex)
-> );
```





### 4.6.3 在已存在的表上创建索引

`CREATE  [UNIQUE | FULLTEXT] INDEX  索引名 ON 表名 (属性名 [(长度)]  [ASC|DESC])；`

**创建普通索引**

//在example表中的id字段上建立名为index7_id的索引

`CREATE INDEX index7_id ON example(id);`

**创建唯一性索引**

需要使用UNIQUE参数来约束

`CREATE UNIQUE INDEX index8_id ON index8(course_id);`

**创建全文索引**

`CREATE FULLTEXT INDEX index9_info ON index9(info);`

**创建单列索引**

//address字段的数据类型为VARCHAR(20)，索引的数据类型为CHAR(4)

`CREATE  INDEX index10_addr ON index10(address(4));`

这样查询数据可以只查询address字段的前4个字段，而不需要全部查询

**创建多列索引**

`CREATE  INDEX index11_na ON index11(name,address);`



### 4.6.4 修改索引

`ALTER TABLE 表名 ADD  [UNIQUE | FULLTEXT] INDEX  索引名 (属性名 [(长度)]  [ASC|DESC])`

**创建普通索引**

//在example表中的id字段上建立名为index7_id的索引

`ALTER TABLE index13 ADD INDEX index13_name(name(20));`

**创建唯一性索引**

需要使用UNIQUE参数来约束

`ALTER TABLE index14 ADD UNIQUE INDEX index14_id(course_id);`

**创建全文索引**

`ALTER TABLE index15 ADD FULLTEXT INDEX index15_info(info);`

**创建单列索引**

`ALTER TABLE index16 ADD INDEX index16_addr(address(4));`

这样查询数据可以只查询address字段的前4个字段，而不需要全部查询

**创建多列索引**

`ALTER TABLE index17 ADD INDEX index17_na(name,address);`



### 4.6.5 删除索引

DROP INDEX 索引名 ON 表名

`DROP INDEX id ON index1;`





# 5.SQL优化

推荐：阿里巴巴Java开发手册里面的SQL规约

## 5.1执行计划

使用**执行计划进行删除修改**并**不会对表产生实际影响**。



MySQ使用 `explain + sq语句`查看 执行计划，该执行计划不一定完全正确但是可以参考。

```
EXPLAIN SELECT * FROM user WHERE nid = 3;
```

![img](https://img2018.cnblogs.com/blog/1334255/201812/1334255-20181218161456575-1277410542.png)

列名 |描述
--|--
id |在一个大的查询语句中每个 SELECT 关键字都对应一个唯一的 id
select_type |SELECT 关键字对应的那个查询的类型
table |表名
partitions| 匹配的分区信息
type |针对单表的访问方法
possible_keys |可能用到的索引
key |实际上使用的索引
key_len| 实际使用到的索引长度
ref |当使用索引列等值查询时，与索引列进行等值匹配的对象信息
rows |预估的需要读取的记录条数
filtered |某个表经过搜索条件过滤后剩余记录条数的百分比
Extra |一些额外的信息



### id

**连接查询**的执行计划中，每个表都会对应一条记录，这些记录的**id列的值是相同**的
出现在前边的表表示驱动表，出现在后边的表表示被驱动表
`EXPLAIN SELECT * FROM s1 INNER JOIN s2`



**包含子查询**的查询语句的执行计划中，每个 SELECT 关键字都会对应一个**唯一的 id 值**
`EXPLAIN SELECT * FROM s1 WHERE key1 IN (SELECT key1 FROM s2) OR key3 = 'a';`



**查询优化器**可能对涉及**子查询**的查询语句进行**重写**，从而**转换为连接查询**

 

UNION 子句的查询语句来说，每个 SELECT 关键字对应一个 id 值，同时有一个id = NULL的临时表，但是对于 UNION AL就不需要为最终的结果集进行去重，也就没有该NULL的临时表

`EXPLAIN SELECT * FROM s1 UNION SELECT * FROM s2`

![image-20220427103042364](img\MySQL\image-20220427103042364.png)

### selct_type

| select_type  |      说明      |
| :----------: | :------------: |
|    SIMPLE    |    简单查询    |
|   PRIMARY    |   最外层查询   |
|   SUBQUERY   |  映射为子查询  |
|   DERIVED    |     子查询     |
|    UNION     |      联合      |
| UNION RESULT | 使用联合的结果 |

1. 查询语句中不包含 UNION 或者子查询的查询都算作是 `SIMPLE` 类型

2. 对于包含 UNION 、 UNION AL或者子查询的大查询来说，它是由几个小查询组成的，其中最左边的那个查询
   的 select_type 值就是 `PRIMARY`

   `EXPLAIN SELECT * FROM s1 UNION SELECT * FROM s2`第一个select为primary

3. 包含 UNION 、 UNION AL或者子查询的大查询来说，除去第一个以外都是`UNION`，当为UNION时会有`UNION RESULT`临时表

4. 查询优化器决定采用将该子查询物化，如果为不相关子查询则为`SUBQUERY`，否则为`DEPENDENT SUBQUERY`

5. 将子查询物化之后与外层查询进行连接查询时 `MATERIALIZED` 



------

**table** : 正在访问的表名

------



### type

|                             type                             |                             说明                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                             ALL                              |                         全数据表扫描                         |
|                            index                             |             索引覆盖，但需要扫描全部的索引记录时             |
|                            RANGE                             |                     对索引列进行范围查找                     |
|                         INDEX_MERGE                          |                合并索引，使用多个单列索引搜索                |
|                             REF                              |                   根据索引查找一个或多个值                   |
|                            EQ_REF                            |             搜索时使用primary key 或 unique类型              |
|                            CONST                             | 常量，表最多有一个匹配行,因为仅有一行,在这行的列值可被优化器剩余部分认为是常数,const表很快,因为它们只读取一次。 |
|                            SYSTEM                            |       系统，表仅有一行(=系统表)。仅MyISAM、Memory适用        |
| 性能：`all` < `index` < `range` < `index_merge` < `ref_or_null` < `ref` < `eq_ref` < `system/const` |             性能在 range 之下基本都可以进行调优              |

------

**possible_keys** : 可能使用的索引

------

**key** : 真实使用的索引

------

**key_len** : MySQL中使用索引字节长度

- 对于使用**固定长度**类型的索引列来说，它实际占用的存储空间的最大长度就是该固定值，对于指定字符集的
  变长类型的索引列来说，比如某个索引列的类型是 VARCHAR(100) ，使用的字符集是 utf8 ，那么该列实际占
  用的最大存储空间就是 100 × 3 = 300 个字节。
- 如果该索引列**可以存储 NULL** 值，则 key_len 比不可以存储 NUL值时**多1个字节**。
- 对于**变长字段**来说，都会有**2个字节的空间**来存储该变长列的实际长度。

`EXPLAIN SELECT * FROM s1 WHERE id = 5;` id为int主键，不可以存null，则为key_len = 4

` EXPLAIN SELECT * FROM s1 WHERE key2 = 5;` key2 为int ，可以为null，则key_len = 5

`EXPLAIN SELECT * FROM s1 WHERE key1 = 'a'`  key1 列的类型是 **VARCHAR(100)** ，所以该列实际最多占用的存储空间就是 300 字节，又因为该列**允许存储NULL** 值，所以 key_len 需要加 1 ，又因为该列是**可变长度列**，所以 key_len 需要加 2 ，所以最后 ken_len 的值就是 303 。

------

**rows** : mysq预估为了找到所需的行而要读取的行数

------

|                    extra                    |                             说明                             |
| :-----------------------------------------: | :----------------------------------------------------------: |
|                 Using index                 |         此值表示mysql将使用覆盖索引，以避免访问表。          |
|                 Using where                 | mysq将在存储引擎检索行后再进行过滤，许多where条件里涉及索引中的列，当(并且如果)它读取索引时，就能被存储引擎检验，因此不是所有带where子句的查询都会显示“Using where”。有时“Using where”的出现就是一个暗示：查询可受益于不同的索引。 |
|               Using temporary               |              mysq对查询结果排序时会使用临时表。              |
|               Using filesort                | mysql会对结果使用一个外部索引排序，而不是按索引次序从表里读取行。mysql有两种文件排序算法，这两种排序方式都可以在内存或者磁盘上完成，explain不会告诉你mysql将使用哪一种文件排序，也不会告诉你排序会在内存里还是磁盘上完成。 |
| Range checked for each record(index map: N) | 没有好用的索引，新的索引将在联接的每一行上重新估算，N是显示在possible_keys列中索引的位图，并且是冗余的 |

**limit**

limit 匹配后就不会继续进行扫描

```
mysql> SELECT * FROM user WHERE emai= 'klvchen123@126.com' LIMIT 1;
+-----+------------+--------------------+-------+
| nid | name       | emai             | extra |
+-----+------------+--------------------+-------+
| 123 | klvchen123 | klvchen123@126.com | 123   |
+-----+------------+--------------------+-------+
1 row in set (0.01 sec)
```



详细参看[博客](https://www.cnblogs.com/ggjucheng/archive/2012/11/11/2765237.html)或[备份](pages/mysql_explain.html)



## 5.2 查看类似优化后语句

在执行完`explain`查询后

使用`SHOW WARNINGS`

 Message 字段展示的信息**类似于查询优化器将我们的查询语句重写后的语句**，**并不是等价于**，也就是说 Message 字段展示的信息并不是标准的查询语句，在很多情况下并不能直接拿到黑框框中运行，它只能作为帮助我们理解查 MySQ将如何执行查询语句的一个参考依据而已



### 追踪执行计划

查看是否开启`SHOW VARIABLES LIKE 'optimizer_trace'`默认不开

完整的使用 `optimizer trace` 功能的步骤总结如下：

1. 打开optimizer trace功能 (默认情况下它是关闭的):
   `SET optimizer_trace="enabled=on";`

2. 这里输入你自己的查询语句
   `SELECT ...;`

3. 从OPTIMIZER_TRACE表中查看上一个查询的优化过程
   `SELECT * FROM information_schema.OPTIMIZER_TRACE;`

4. 可能你还要观察其他语句执行的优化过程，重复上边的第2、3步

​			...

5. 当你停止查看语句的优化过程时，把optimizer trace功能关闭
   `SET optimizer_trace="enabled=off";`





## 5.3 正确使用索引

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
# 若 name 和 emai组成组合索引
create index ix_name_emaion user(name,email);

name and emai-- 使用索引
emaiand name -- 不使用索引
name                  -- 使用索引
emai                 -- 不使用索引
```

`没用使用索引`
![img](https://img2018.cnblogs.com/blog/1334255/201812/1334255-20181219135548420-1350585399.png)
![img](https://img2018.cnblogs.com/blog/1334255/201812/1334255-20181219135612019-804744999.png)



## 5.4 慢查询分析

`show variables like 'slow_query_log'  `//查看是否开启慢查询日志

`set globaslow_query_log_file=' /usr/share/mysql/sql_log/mysql-slow.log' `//慢查询日志的位置

`set globalog_queries_not_using_indexes=on;`//开启慢查询日志

`set globalong_query_time=1; `//大于1秒钟的数据记录到慢日志中，如果设置为默认0，则会有大量的信息存储在磁盘中，磁盘很容易满掉

![](C:\java\markdown\docs\img\MySQL\image-20220428151304982.png)

慢查询格式

​	1、# Time: 180526  1:06:54 -------查询的执行时间

​	2、# User@Host: root[root] @ localhost []  Id:   4 -------执行sql的主机信息

​	3、# Query_time: 0.000401  Lock_time: 0.000105 Rows_sent: 2  Rows_examined: 2-------SQL的执行信息：

​			Query_time：SQL的查询时间

​			Lock_time：锁定时间

​			Rows_sent：所发送的行数

​			Rows_examined：锁扫描的行数

​	4、SET timestamp=1527268014; -------SQL执行时间

​	5、select * from staff; -------SQL的执行内容



## 5.5 具体案例分析

### 5.5.1 函数Max()的优化

用途：查询最后支付时间-优化max（）函数

语句：`select max(payment_date) from payment;`

执行计划：

`explain select max(payment_date) from payment;`

![img](C:\java\markdown\docs\img\MySQL\wpsAF45.tmp.jpg) 

<img src="C:\java\markdown\docs\img\MySQL\wpsAF46.tmp.jpg" alt="img" style="zoom:67%;" /> 

 

可以看到显示的执行计划，并不是很高效，可以拖慢服务器的效率，如何优化了？

**优化**：创建索引`create index inx_paydate on payment(payment_date);`

<img src="C:\java\markdown\docs\img\MySQL\wpsAF47.tmp.jpg" alt="img" style="zoom:67%;" /> 

 

<img src="C:\java\markdown\docs\img\MySQL\wpsAF48.tmp.jpg" alt="img" style="zoom:67%;" /> 

 

索引是顺序操作的，不需要扫描表，执行效率就会比较恒定，

 

### 5.5.2 函数Count()的优化

需求：在一条SQL中同时查处2006年和2007年电影的数量

错误的方式：`select count(release_year='2006' or release_year='2007') from film;`

![img](C:\java\markdown\docs\img\MySQL\wpsAF49.tmp.jpg) 

2006和2007年分别是多少，判断不出来

` select count(*) from film where release_year='2006' or release_year='2007'; `

![img](C:\java\markdown\docs\img\MySQL\wpsAF4A.tmp.jpg) 

正确的编写方式：

`select count(release_year='2006' or null) as '06films',count(release_year='2007' or null) as '07films' from film;`

![img](C:\java\markdown\docs\img\MySQL\wpsAF4B.tmp.jpg) 



#### count(*)和count(id)区别

结论：

`count(*), count(1), count(常数)`含义相同：会统计`NULL`。同时`count(主键id)`也可以统计表行数。因为主键不可以为`NULL`。

`count(id), count(列名)`含义相同：不会统计`NULL`

`count(distinct col)` 计算该列`除 NULL 之外`的不重复行数

`count(distinct id1, id2)`会统计id1和id2不同的列，有一列出现NULL则不会统计

```mysql
mysql> select * from t1;
+------+------+
| id   | Name |
+------+------+
|    1 | haha |
|    2 | gaga |
| NULL | kaka |
| NULL | NULL |
|    5 | NULL |
+------+------+
5 rows in set (0.00 sec)

mysql> select count(distinct id, name) from t1;
+--------------------------+
| count(distinct id, name) |
+--------------------------+
|                        2 |
+--------------------------+
1 row in set (0.00 sec)

mysql> select count(distinct id) from t1;
+--------------------+
| count(distinct id) |
+--------------------+
|                  3 |
+--------------------+
```



**原理**：

首先对查看下执行计划后看具体执行的语句

```mysql
explain select count(*) from t1;
show warnings;
+-------+------+---------------------------------------------------------------+
| Level | Code | Message                                                       |
+-------+------+---------------------------------------------------------------+
| Note  | 1003 | /* select#1 */ select count(0) AS `count(*)` from `test`.`t1` |
+-------+------+---------------------------------------------------------------+

mysql> explain select count(1) from t1;
mysql> show warnings;
+-------+------+---------------------------------------------------------------+
| Level | Code | Message                                                       |
+-------+------+---------------------------------------------------------------+
| Note  | 1003 | /* select#1 */ select count(1) AS `count(1)` from `test`.`t1` |
+-------+------+---------------------------------------------------------------+
```

查看`count(*)`时，等同于查看`count(0)`

![img](C:\java\markdown\docs\img\MySQL\d4e8294b3a544c3586b576199989b6eftplv-k3u1fbpfcp-zoom-in-crop-mark1304000.webp)

查询判断时会选择成本低的索引或者直接全表扫描来找到符合条件的值，然后每符合就`+1`



#### 超长表行数统计

`MyISAM` 引擎把**一个表的总行数存在了磁盘上**，因此执行 count(*) 的时候会直接返回这个数，
效率很高；
而 `InnoDB` 引擎就麻烦了，它执行 count(*) 的时候，需要把数据**一行一行地从引擎里面读出**
来，然后累积计数

解决方案1：用缓存系统保存计数

但是存在丢失风险

不及时更新导致的不准确，多线程下不准确



方案2：在数据库保存计数

利用单独的表记录行数，每次更新删除操作和更改该行数放在同一个事务里面，保证准确。





### 5.5.3 group by的优化

最好使用同一表中的列，

需求：每个演员所参演影片的数量-（影片表和演员表） 

`explain select actor.first_name,actor.last_name,count(*)from sakila.film_actorinner join sakila.actor using(actor_id)group by film_actor.actor_id;`

![img](C:\java\markdown\docs\img\MySQL\wps99F2.tmp.jpg) 

 

优化后的SQL：

`explain select actor.first_name,actor.last_name,c.cntfrom sakila.actor inner join (select actor_id,count(*) as cnt from sakila.film_actor group by actor_id)as c using(actor_id);`

![img](C:\java\markdown\docs\img\MySQL\wps99F3.tmp.jpg) 

<img src="C:\java\markdown\docs\img\MySQL\wps99F4.tmp.jpg" alt="img" style="zoom:67%;" /> 

说明：从上面的执行计划来看，这种**优化后的方式没有使用临时文件和文件排序**的方式了，取而代之的是使用了索引。查询效率老高了。

这个时候我们表中的数据比较大，会大量的占用IO操作，优化了sql执行的效率，节省了服务器的资源，因此我们就需要优化。

注意：

1、mysq中using关键词的作用：也就是说要使用using,那么表a和表b必须要有相同的列。

2、在用Join进行多表联合查询时，我们通常使用On来建立两个表的关系。其实还有一个更方便的关键字，那就是Using。

3、如果**两个表的关联字段名是一样的**，就可以**使用Using**来建立关系，简洁明了。



#### 原理

https://juejin.cn/post/6973300662738092069#heading-2

1. **全字段排序**：MySQL 会给每个查询线程分配一块小**内存**，用于**排序**的，称为 **sort_buffer**，将查询所需的字段全部读取到sort_buffer中

2. **磁盘临时文件辅助排序**：实际上，sort_buffer的大小是由一个参数控制的：**sort_buffer_size**。如果要排序的数据小于sort_buffer_size，排序在**sort_buffer** 内存中完成，如果要排序的数据大于sort_buffer_size，则**借助磁盘文件来进行排序**

```mysql
## 打开optimizer_trace，开启统计
set optimizer_trace = "enabled=on";
## 执行SQL语句
select name,age,city from staff where city = '深圳' order by age limit 10;
## 查询输出的统计信息
select * from information_schema.optimizer_trace 
```

使用了磁盘临时文件，整个排序过程又是怎样的呢？

1. 从**主键Id索引树**，拿到需要的数据，并放到**sort_buffer内存**块中。当sort_buffer快要满时，就对sort_buffer中的数据排序，排完后，把数据临时放到磁盘一个小文件中。
2. 继续回到主键 id 索引树取数据，继续放到sort_buffer内存中，排序后，也把这些数据写入到磁盘临时小文件中。
3. 继续循环，直到取出所有满足条件的数据。最后把磁盘的临时排好序的小文件，合并成一个有序的大文件。

**TPS:** 借助磁盘临时小文件排序，实际上使用的是**归并排序**算法。



3. **rowid 排序**：只把查询SQL**需要用于排序的字段和主键id**，放到sort_buffer中。那怎么确定走的是全字段排序还是rowid 排序排序呢？
   实际上有个参数控制的。这个参数就是**max_length_for_sort_data**，它表示MySQL用于排序行数据的长度的一个参数，如果单行的长度超过这个值，MySQL 就认为单行太大，就换rowid 排序。我们可以通过命令看下这个参数取值。

```mysql
show variables like 'max_length_for_sort_data';
```

**max_length_for_sort_data** 默认值是1024。因为本文示例中name,age,city长度=64+4+64 =132 < 1024, 所以走的是全字段排序。我们来改下这个参数，改小一点，

```mysql
## 修改排序数据最大单行长度为32
set max_length_for_sort_data = 32;
```

sort_buffer可以放更多数据，但是需要再回到原表去取数据，比全字段排序多一次**回表**



#### 常用优化

对于InnoDB存储引擎，会优先使**用全字段**排序。可以发现 **max_length_for_sort_data** 参数设置为1024，这个数比较大的。一般情况下，排序字段不会超过这个值，也就是都会走**全字段**排序。

**调整参数优化**

我们还可以通过调整参数，去优化order by的执行。比如可以调整sort_buffer_size的值。因为sort_buffer值太小，数据量大的话，会借助磁盘临时文件排序。如果MySQL服务器配置高的话，可以使用稍微调整大点。

我们还可以调整max_length_for_sort_data的值，这个值太小的话，order by会走rowid排序，会回表，降低查询性能。所以max_length_for_sort_data可以适当大一点。



**没有where条件的order by**

那么，这时候order by后面的字段是否需要加索引呢。如有这么一个SQL，create_time是否需要加索引：

```mysql
select * from A order by create_time;
```

**无条件查询的话，即使create_time上有索引,也不会使用到**。因为**MySQL优化器认为走普通二级索引，再去回表成本比全表扫描排序更高**。所以选择走全表扫描,然后根据全字段排序或者rowid排序来进行。

如果查询SQL修改一下：

```mysql
select * from A order by create_time limit m;
```

无条件查询,如果m值较小,是可以走索引的.因为MySQL优化器认为，根据索引有序性去回表查数据,然后得到m条数据,就可以终止循环,那么成本比全表扫描小,则选择走二级索引。



**混用排序索引**

MySQL是8.0版本，支持**Descending Indexes**，可以这样修改索引

`alter table t1 add Index ix1(a asc, b desc);`





 

### 5.5.4 Limit查询的优化

Limit常用于分页处理，时长会伴随order by从句使用，因此大多时候回使用Filesorts这样会造成大量的IO问题。



需求：查询影片id和描述信息，并根据主题进行排序，取出从序号50条开始的5条数据。

`select film_id,description from sakila.film order by title limit 50,5;`

执行的结果：

![img](C:\java\markdown\docs\img\MySQL\wps99F5.tmp.jpg) 

在查看一下它的执行计划：

![img](C:\java\markdown\docs\img\MySQL\wps99F6.tmp.jpg) 

对于这种操作，我们该用什么样的优化方式了？

该表已经按照`title有序`插入，与id增加一致，所以按照`id主键`排序和按照`title`排序一样

#### 优化1

使用有索引的列或主键进行order by操作，因为大家知道，innodb是按照主键的逻辑顺序进行排序的。可以避免很多的IO操作。

`select film_id,description from sakila.film order by film_id limit 50,5;`

![img](C:\java\markdown\docs\img\MySQL\wps99F7.tmp.jpg) 

查看一下执行计划

![img](C:\java\markdown\docs\img\MySQL\wps99F8.tmp.jpg) 

那如果我们获取从500行开始的5条记录，执行计划又是什么样的了？

`explain select film_id,description from sakila.film order by film_id limit 500,5\G`

![img](C:\java\markdown\docs\img\MySQL\wps9A08.tmp.jpg) 

 

![img](C:\java\markdown\docs\img\MySQL\wps9A09.tmp.jpg) 

随着我们翻页越往后，IO操作会越来越大的，如果一个表有几千万行数据，翻页越后面，会越来越慢，因此我们要进一步的来优化。

 

#### 优化2

记录上次返回的主键， 在**下次查询时使用主键过滤**。（说明：避免了数据量大时扫描过多的记录）

上次limit是50,5的操作，因此我们在这次优化过程需要使用上次的索引记录值，

`select film_id,description from sakila.film  where film_id >55 and film_id<=60 order by film_id limit 1,5;`

查看执行计划：

![img](C:\java\markdown\docs\img\MySQL\wps9A0A.tmp.jpg) 

![img](C:\java\markdown\docs\img\MySQL\wps9A0B.tmp.jpg) 

![img](C:\java\markdown\docs\img\MySQL\wps9A0C.tmp.jpg) 

结论：**扫描行数不变，执行计划是很固定，效率也是很固定的**

注意事项：

主键要顺序排序并连续的，如果主键中间空缺了某一列，或者某几列，会出现列出数据不足5行的数据；如果不连续的情况，建立一个附加的列index_id列，保证这一列数据要自增的，并添加索引即可。



但是有时候分页id并不是连续的

#### **优化3**

`film_info`有二级索引`idx1(film_id)`，

先子查询通过二级索引定位需要找到位置的`film_id`，然后通过等值查询去聚集索引回表10次查询。

这样避免了直接二级索引回表500多次。

```mysql
mysql> explain select t1.* from film_info as t1, (select film_id from film_info order by film_id limit 500,10 ) as t2 where t1.film_id = t2.film_id \G
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: <derived2>
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 510
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 1
  select_type: PRIMARY
        table: t1
   partitions: NULL
         type: ref
possible_keys: idx1
          key: idx1
      key_len: 2
          ref: t2.film_id
         rows: 1
     filtered: 100.00
        Extra: NULL
*************************** 3. row ***************************
           id: 2
  select_type: DERIVED
        table: film_info
   partitions: NULL
         type: index
possible_keys: NULL
          key: idx1
      key_len: 2
          ref: NULL
         rows: 510
     filtered: 100.00
        Extra: Using index
```







### 5.5.5 Delete/update 优化

对于delete in删除默认没有优化

两张表都声明了`name`为索引

`delete from account where name in (select name from old_account);`

可以看出没有使用索引去删除

![img](C:\java\markdown\docs\img\MySQL\42febbf243d24c62afc9e4e1c8ce2cabtplv-k3u1fbpfcp-zoom-in-crop-mark1304000.webp)

解决方案：

默认`select in`子查询会优化为`semi in` 子查询

![img](C:\java\markdown\docs\img\MySQL\62bc02bfc04b4ba5af07c1ad9f3c40c4tplv-k3u1fbpfcp-zoom-in-crop-mark1304000.webp)



既然没有自动优化，那就手动修改为`join查询`

![img](C:\java\markdown\docs\img\MySQL\2f4c8365a4e64dfb82a7b3152e5a392dtplv-k3u1fbpfcp-zoom-in-crop-mark1304000.webp)

对于update或者delete子查询的语句，**MySQL官网**也是推荐join的方式优化



解决方案二：其实呢，给表加别名，也可以解决这个问题哦，如下:

```mysql
explain delete a from account as a where a.name in (select name from old_account)
```

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d25ff4209ae4e44bcda8b6eb32440de~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

**为什么加别个名就可以走索引了呢？**

**what**？为啥加个别名，delete in子查询又行了，又走索引了？

我们回过头来看看explain的执行计划，可以发现Extra那一栏，有个**LooseScan**。 ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e954e1a091654813bea0f8b65e7b3a48~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

**LooseScan是什么呢？** 其实它是一种策略，是**semi join子查询**的一种执行策略。

> 因为子查询改为join，是可以让delete in子查询走索引;而**加别名**，会走**LooseScan策略**，而LooseScan策略，本质上就是**semi join子查询**的一种执行策略。

因此，加别名就可以让delete in子查询走索引啦！



### 5.5.6 防止隐式类型转化

`name` 是varchar字符串类型，对那么添加索引
`select * from t1 where name =123;`

```mysql
mysql> alter table t1 add index idx1(name);
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

# 将数字123隐式转换为字符串，使用了全表扫描TYPE = ALL ，没有用索引
mysql> explain select * from t1 where name = 123;
+----+-------------+-------+------------+------+---------------+------+
| id | select_type | table | partitions | type | possible_keys | key  |
+----+-------------+-------+------------+------+---------------+------+
|  1 | SIMPLE      | t1    | NULL       | ALL  | idx1          | NULL |
+----+-------------+-------+------------+------+---------------+------+
1 row in set, 3 warnings (0.00 sec)

# 没有转换时，使用索引type = ref , 使用了二级索引的等值匹配
mysql> explain select * from t1 where name = '123';
+----+-------------+-------+------------+------+---------------+------+
| id | select_type | table | partitions | type | possible_keys | key  |
+----+-------------+-------+------------+------+---------------+------+
|  1 | SIMPLE      | t1    | NULL       | ref  | idx1          | idx1 |
+----+-------------+-------+------------+------+---------------+------+
```









## 5.6 优化插入记录

插入记录时，索引、唯一性校验都会影响到插入记录的速度。而且，一次插入多条记录和多次插入记录所耗费的时问是不一样的。根据这些情况，分别进行不同的优化。

- **禁用索引**

  插入记录时，MySQL会根据表的索引对插入的记录进行排序。如果插入大量数据时，这些排序会降低插入记录的速度。为了解决这种情况，在插入记录之前先禁用索引。等到记录都插入完毕后再开启索引。禁用索引的语句如下：

  `ALTER TABLE 表名 DISABLE KEYS；`

  重新开启索引的语句如下：

  `ALTER TABLE 表名 ENABLE KEYS：`

  对于新创建的表，可以先不创建索引。等到记录都导入以后再创建索引。这样可以提高导入数据的速度。

- **禁用唯一性检查**

插入数据时，MySQL会对插入的记录进行唯一性校验。这种校验也会降低插入记录的速度。可以在插入记录之前禁用唯一性检查。等到记录插入完毕后再开启。

禁用唯一性检查的语法如下：

`SET UNIQUE_CHECKS=0；`

重新开启唯一性检查的语句如下：

`SET UNIQUE_CHECKS=1；`

- **优化INSERT语句**

  插入多条记录时，可以采取两种写INSERT语句的方式。第一种是一个INSERT语句插入多条记录。第二种是一个INSERT语句只插入一条记录，执行多个INSERT语句来插入多条记录。第一种方式减少了与数据库之问的连接等操作，其速度比第二种方式要快。

技巧：当插入大量数据时，建议使用一个INSERT语句插入多条记录的方式。而且，如果能用LOAD DATA INFILE语句，就尽量用LOAD DATA INFILE语句。因为LOAD DATA INFILE语句导入数据的速度比1NSERT语句的速度快。





# 6. 流程分析



![](img\MySQL\167f4c.awebp)

**1) 第一层负责连接处理，授权认证，安全等等**

- 每个客户端连接都会在服务器进程中拥有一个线程，服务器维护了一个线程池，因此不需要为每一个新建的连接创建或者销毁线程。
- 当客户端连接到Mysql服务器时，服务器对其进行认证，通过用户名和密码认证，也可以通过SSL证书进行认证。
- 一旦客户端连接成功，服务器会继续验证客户端是否具有执行某个特定查询的权限。

**2）第二层负责编译并优化SQL**

- 这一层包括查询解析，分析，优化，缓存以及所有的的内置函数。
- 对于SELECT语句，在解析查询前，服务器会先检查查询缓存，如果能在其中找到对应的查询结果，则无需再进行查询解析、优化等过程，直接返回查询结果。
- 所有跨存储引擎的功能都在这一层实现:存储过程，触发器，视图。

**3）第三层是存储引擎。**

- 存储引擎负责在MySQL中存储数据、提取数据。
- 存储引擎通过API与上层进行通信，这些API屏蔽了不同存储引擎之间的差异，使得这些差异对上层查询过程透明。
- 存储引擎不会去解析SQL，不同存储引擎之间也不会相互通信，而只是简单地响应上层服务器的请求。




**客户端连接服务端**

通过`TCP/IP`加`端口`连接 `mysq-h127.0.0.1 -uroot -P3306 -p`

 也可以通过命名管道或共享内存 、 Unix域套接字



**查询缓存**

如果**两个查询请求在任何字符上的不同**（例如：空格、注释、大小写），都会导致**缓存不会命中**。

另外，如果查询请求中包含某些系统函数、用户自定义变量和函数、一些系统表，如mysq、information_schema、 performance_schema 数据库中的表，那这个**请求就不会被缓存**。

以某些系统函数举例，可能同样的函数的两次调用会产生不一样的结果，比如函数 NOW ，每次调用都会产生最新的当前时间，如果在一个查询请求中调用了这个函数，那即使查询请求的文本信息都一样，那不同时间的两次查询也应该得到不同的结果，如果在第一次查询时就缓存了，那第二次查询的时候直接使用第一次查询的结果就是错误的！
不过既然是缓存，那就有它缓存失效的时候。MySQL的缓存系统会监测涉及到的每张表，**只要该表的结构或者数据被修改，如对该表使用了 INSERT** 、 UPDATE 、 DELETE 、 TRUNCATE TABLE 、 ALTER TABLE 、 DROP TABLE 或
DROP DATABASE 语句，那使用**该表的所有高速缓存查询都将变为无效并从高速缓存中删除**！

虽然查询缓存有时可以提升系统性能，但也不得不因维护这块缓存而造成一些开销，比如每次都要去查
询缓存中检索，查询请求处理完需要更新查询缓存，维护该查询缓存对应的内存区域。

**从MySQ5.7.20开始，不推荐使用查询缓存，并在MySQ8.0中删除**。



# 7. 修改字符集

修改**服务器**默认字符集为`utf8`

进入`C:\ProgramData\MySQL\MySQServer 5.7`修改`my.ini`

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
 
 CREATE table t ( coVARCHAR(20) );
 
 INSERT INTO t(col) VALUES ('a'), ('b'), ('B'), ('A'), ('我');
 
 select * from t order by coasc;
 
 ALTER TABLE t MODIFY coVARCHAR(20) COLLATE utf8_bin;
```



## 大小写问题

MySQL在Linux下数据库名、表名、列名、别名大小写规则是这样的：
　　 1、数据库名与表名是严格区分大小写的；
　　 2、表的别名是严格区分大小写的；
　　 3、**列名与列的别名在所有的情况下均是忽略大小写**的；
　　 4、变量名也是严格区分大小写的；

```mysql
mysql> show variables like '%lower%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| lower_case_file_system | OFF   |
| lower_case_table_names | 0     |
+------------------------+-------+
lower_case_file_system是一个只读参数，无法被修改，
这个参数是用来告诉你在当前的系统平台下,是否对文件名大小写敏感
```



Windows下

```mysql
mysql> show variables like '%lower%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| lower_case_file_system | ON    |
| lower_case_table_names | 1     |
+------------------------+-------+
2 rows in set, 1 warning (0.00 sec)
```





> 阿里巴巴规范：
>
> 【强制】表名、字段名必须使用**小写字母或数字** ， 禁止出现数字开头，禁止两个下划线中间只
> 出现数字。数据库字段名的修改代价很大，因为无法进行预发布，所以字段名称需要慎重考虑。
> 说明：MySQL 在 Windows 下不区分大小写，但在 Linux 下默认是区分大小写。因此，数据库名、表名、
> 字段名，都不允许出现任何大写字母，避免节外生枝。
> 正例：aliyun_admin，rdc_config，level3_name
> 反例：AliyunAdmin，rdcConfig，level_3_name



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

变长字段长度列表中**只存储值为 非NUL的列内容占用的长度**，值为 NUL的列的长度是不储存的 

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
   如果该 VARCHAR 类型的列没有 NOT NUL属性，那最多只能存储 65532 个字节的数据，因为真实数据的长度可能占用2个字节， NUL值标识需要占用1个字节

   如果 VARCHAR 类型的列有 NOT NUL属性，那最多只能存储 65533 个字节的数据，因为真实数据的长度可能占用2个字节，不需要 NUL值标识

2.  gbk 字符集表示一个字符最多需要 2 个字节，那在该字符集下， M 的最大取值就是 32766 （也就是：65532/2），也就是说最多能存储 32766 个字符；

3. utf8 字符集表示一个字符最多需要 3 个字节，那在该字符集下， M 的最大取值就是 21844 ，就是说最多能存
   储 21844 （也就是：65532/3）个字符

### 行溢出分页

一个页的大小一般是 16KB ，也就是 16384 字节，而一个 VARCHAR(M) 类型的列就最多可以存储 65532 个字节，这样就可能造成一个页存放不了一条记录的尴尬情况

在 **Compact 和 Reduntant** 行格式中，在 记录的真实数据 处只会存储该列的一部分数据，把剩余的数据分散存储在几个其他的页中，然后 **记录的真实数据 处用20个字节存储指向这些页的地址**

在本记录的真实数据处只会存储该列的前 768 个字节的数据和一个指向其他页的地址，然后把剩下的数据存放到其他页中，这个过程也叫做 **行溢出** ，存储超出 768 字节的那些页面也被称为 **溢出页** 



不只是 **VARCHAR(M)** 类型的列，其他的 **TEXT、BLOB** 类型的列在存储数据非常多的时候也会发生 **行溢出** 。



MySQ中规定一个页中至少存放两行记录



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









# 数据目录



数据库`info`下包含文件

> 2022/04/24  14:57                61   db.opt     # 该数据库的属性，比如字符集、比较规则
> 2022/04/24  15:00             8,586  test.frm  # test表的结构
> 2022/04/24  15:00            98,304 test.ibd  # test表的数据和索引





MyISAM数据目录

> 2022/04/24  15:22             8,586 user.frm
> 2022/04/24  15:22                 0 user.MYD
> 2022/04/24  15:22             1,024 user.MYI



视图表示

`视图名.frm`



表空间中的每一个页都对应着一个页号，也就是 FIL_PAGE_OFFSET ，这个页号由4个字节组成，也就是32个比特位，所以一个表空间最多可以拥有2³²个页，如果按照页的默认大小16KB来算，一个表空间最多支持64TB的数据。





# B+索引

## 适用条件

​						建立索引为`（a,b,c）`

**全值匹配**：搜索条件中的列和索引列一致，如`where a = 10 and b = 20 and c = 30`

**匹配左边的列**：搜索条件中的各个列必须是联合索引中从最左边连续的列，建立索引为`（a,b,c）`则搜索条件`a=10 and b=20` 从左边`a`开始匹配，当`a`匹配成功后，再去匹配`b`

**匹配列前缀**：匹配字符串的前几个字符`name LIKE 'As%'`

**匹配范围值**：`where a > 10 and a < 20`
		`where a > 10 and b > 20`如果对多个列同时进行范围查找的话，只有对索引最左边的那个列进行范围查找的时候才能用到 B+ 树索引，此时只有`a > 10`用到了索引，而所有`a>10`的数中`b`并不是有序的，只有当`a=10`时`b`才是有序。

**精确匹配某一列并范围匹配另外一列**：`where a = 10 and b > 20 and c > 30` 对`a=10`精确匹配，对于`b>20`范围匹配，但是`c>30`就无法使用索引了

**用于排序**：`order by a,b,c`与索引顺序一致且不存在ASC，DESC混用时可以使用索引，`where a = 10 order by b,c`匹配`a=10`后`b,c`与索引顺序匹配且有序

但是`order by a, b DESC`混用不行，`order by a, e`出现了非索引列`e`不行，`order by upper(a)`使用函数修饰不行

**用于分组**：`select count(*) from table1 group by a,b`分组与索引顺序一致就可以



## 回表

使用二级索引时，如果搜索结果需要列二级索引中没有，则需要根据二级索引检索出来的主键逐一去聚簇索引中寻找。

二级索引使用顺序I/O查找，性能好

但是聚簇索引使用随机I/O，性能差



优化器会决定采用 `全表扫描` 还是 `二级索引 + 回表`



### 覆盖索引

为了避免回表，也就是二级索引中添加没有包含的列，比如需要查看`(a,b,c,e)`那就添加索引`(a,b,c,e)`覆盖搜索条件



## 添加索引注意

只为**用于搜索、排序或分组的列**创建索引
为列的**基数大**的列创建索引`（不一样的多）`
索引列的**类型尽量小**`（ TINYINT  < MEDIUMINT  <  INT <  BIGINT）`
可以只对**字符串值的前缀**建立索引`(index (name(5), address(10)) )`
只有索引列在比较表达式中单独出现才可以适用索引 `where a * 10 > 100 不行` `，where a > 100/10 行`
为了尽可能少的让 聚簇索引 发生页面分裂和记录移位的情况，建议让`主键拥有 AUTO_INCREMENT` 属性。
定位并删除表中的重复和冗余索引
尽量使用 **覆盖索引** 进行查询，避免 回表 带来的性能损耗。



# 单表访问方法

 MySQ执行查询语句的方式称之为 访问方法，同一个语句可以有多个方法执行，但是性能不同

## const

只能在**主键**列或者**唯一二级索引**列和**一个常数(不包含NULL)**进行**等值**比较

`select * from user where id = 1`

## ref

普通的二级索引来说，通过索引列进行**等值比较**后可能匹配到**多条连续**的记录

`select * from user where score = 80`

可以包含`NULL`

`select * from user where id is NULL`

包含多个索引列的二级索引来说，只要是**最左边的连续索引列**是与**常数的等值比较**就可能采用 ref的访问方法

`select * from user where a=10 and b = 20`

但是出现了范围匹配就不是了

`select * from user where a = 10 and b > 20`

## ref_or_null

同时包含二级索引的等值比较 和 NULL比较

`select * from user where id=10 or id is NULL`



## eq_ref

**连接查询**时，如果**被驱动表**是通过**主键**或者**唯一二级索引列等值匹配**的方式进行访问的（如果该主键或者
唯一二级索引是**联合索引**的话，**所有的索引列**都必须进行**等值比较**）

对该**被驱动表**的访问方法就是`eq_ref` 

`SELECT * FROM s1 INNER JOIN s2 ON s1.id = s2.id`

## range

无论时**二级**索引还是**聚簇**索引采用了 **范围比较**

`select * from user where id > 10 and score in (70,80)`



## index

遍历二级索引记录的执行方式

有一个二级索引`(a,b,c)`，默认聚簇索引包含`(a,b,c,d,e)`

`select a,b,c from t1 where b = 20`

使用了二级索引搜索列和索引列相同（**覆盖索引**），但是不符合最左匹配原则，此时**遍历整个二级索引**，而不是遍历聚簇索引，因为二级索引包含的数据量少，没有隐藏列和其他列。

## all

直接全表扫描聚簇索引



## index merge

使用`and`连接多个条件时，将多个条件的集合**取交集**



读取多个二级索引之后取交集成本：回表次数少且无需再过滤
按照不同的搜索条件分别读取不同的二级索引
将从多个二级索引得到的主键值取交集，然后进行回表操作



只读取一个二级索引的成本：
按照某个搜索条件读取一个二级索引
根据从该二级索引得到的主键值进行回表操作，然后再过滤其他的搜索条件



比如有几个索引`index key1(a), index key2(b,c)`，默认索引主键为`id primary key`

**情况一**：二级索引列是**等值匹配**的情况，对于**联合索引**来说，在**联合索引中的每个列都必须等值匹配**，不能
出现只出现匹配部分列的情况

`select * from t1 where a = 10 and b = 20 and c = 30`

先找出`索引key1`对应的主键id，然后找出`索引key2`对应的主键id，然后取交集得最后去聚簇索引搜索的主键id

但是如果联合索引不完整就无法使用，此时索引key2只使有了`b`，那么无法合并

`select * from t1 where a = 10 and b = 20`



**更优解**：添加一个新的联合索引覆盖`a,b,c`，那么就减少了一次B+树搜索过程

`alter table t1 add index key_abc(a,b,c), drop key1,key2`



**情况二**：主键列可以是范围匹配，根据二级索引查询出的结果集是按照主键值排序的

`select * from t1 where id > 100 and a = 10`

根据`key1`索引，先找出`a=10`，然后二级索引中主键`id`是有序排序



但是**非主键有序**的就不满足

`select * from t1 where id > 100 and b = 20`

`select * from t1 where a > 100 and b = 20 and c = 30`





## range区间

cond1 AND cond2 ：只有当 cond1 和 cond2 都为 TRUE 时整个表达式才为 TRUE 。
cond1 OR cond2 ：只要 cond1 或者 cond2 中有一个为 TRUE 整个表达式就为 TRUE 。



## Union合并

使用`OR`连接多个条件时，取多个集合并集



比如有几个索引`index key1(a), index key2(b,c), index key3(d)`，默认索引主键为`id primary key`

情况一：二级索引列是等值匹配的情况，对于联合索引来说，在联合索引中的每个列都必须等值匹配，不能
出现只出现匹配部分列的情况。

`select * from t1 where a = 10 or (b=20 and c = 30)`

情况二：主键列可以是范围匹配

`select * from t1 where id > 100 or a = 10`

情况三：使用 Intersection 索引合并的搜索条件

 `select * from t1 where b=20 and c=30 or ( a=10 and d=40 )`

先对`a=10 and d=40`使用交集得到集合，然后通过`or`对两集合取并集



### Sort-Union索引合并

`select * from t1 where a > 10 or d > 20`

先根据二级索引`key1`取出集合，再将主键值排序

然后根据二级索引`key2`取出集合，再将主键值排序

最后将两个集合Union合并，取出来的主键值也是有序的，那么`回表查询时顺序I/O`消耗也不会很大。



但是没有`Sort-Intersection`，取交集一般是两个集合太大取共同点，那么对两个单独集合主键值排序消耗太大不划算。取并集一般是两个集合太小了，那么排序消耗不大可以取交集。





# Join连接

<img src="img\MySQL\sql-join.png" style="zoom: 67%;" />

## 本质

连接 的本质就是把各个连接表中的记录都取出来**依次匹配**的组合**加入结果集**并返回给用户



## 过程

`SELECT * FROM t1, t2 WHERE t1.m1 > 1 AND t1.m1 = t2.m2 AND t2.n2 < 'd'`

在这个查询中我们指明了这三个过滤条件：
`t1.m1 > 1`
`t1.m1 = t2.m2`
`t2.n2 < 'd'`

1. 首先确定第一个需要查询的表，这个表称之为 驱动表
   对驱动表查询`t1.m1 > 1`得到一个结果集A

   ![image-20220426084328961](img\MySQL\image-20220426084328961.png)

2. 根据 t1 表中的记录去找 t2 表中的记录，所以 t2 表也可以被称之为 被驱动表 

   接着查询第二个条件` t1.m1 = t2.m2`
   ![image-20220426084751346](img\MySQL\image-20220426084751346.png)



### 嵌套循环连接

```java
for each row in t1 { 
    #此处表示遍历满足对t1单表查询结果集中的每一条记录
	for each row in t2 { 
        #此处表示对于某条t1表的记录来说，遍历满足对t2单表查询结果集中的每一条记录
		for each row in t3 { 
            #此处表示对于某条t1和t2表的记录组合来说，对t3表进行单表查询
			if row satisfies join conditions, send to client
		}
	}
}
```

这个过程就像是一个嵌套的循环，所以这种**驱动表只访问一次，但被驱动表却可能被多次访问**，访问次数取决于
对驱动表执行单表查询后的结果集中的记录条数的连接执行方式称之为 **嵌套循环连接 （ Nested-Loop Join ）**





## 连接查询成本

**连接查询总成本 = 单次访问驱动表的成本 + 驱动表扇出数 x 单次访问被驱动表的成本**

连接查询成本占大头的其实是 **驱动表扇出数 x 单次访问被驱动表的成本**



优化

- 尽量减少**驱动表**的扇出
- 对**被驱动表**的访问成本尽量低，尽量在被驱动表的连接列上建立索引



对于**内连接**可以选取那个**成本更低的连接顺序**以及该连接顺序下各个表的最优访问方法作为最终的查询计划

- s1 连接 s2 ，也就是 s1 作为驱动表， s2 作为被驱动表。
- s2 连接 s1 ，也就是 s2 作为驱动表， s1 作为被驱动表。





# 优化器

## 条件优化

1. 移除不必要的括号
   `((a = 5 AND b = c) OR ((a > c) AND (c < 5)))` -> `(a = 5 and b = c) OR (a > c AND c < 5)`

2. 常量传递（constant_propagation）
   `a = 5 AND b > a` -> `a = 5 AND b > 5`

3. 等值传递（equality_propagation）
   `a = b and b = c and c = 5` -> `a = 5 and b = 5 and c = 5`

4. 移除没用的条件（trivial_condition_removal）
   `(a < 1 and b = b) OR (a = 6 OR 5 != 5)` -> `a<1 OR a=6`

5. 表达式计算，该列需单独存在
   `a = 5 + 1` -> `a = 6`
   但是`abs(a) > 5` 或`-a < 5`不会优化列

6. HAVING子句和WHERE子句的合并
   如果查询语句中没有出现诸如 SUM 、 MAX 等等的聚集函数以及 GROUP BY 子句，优化器就把 HAVING 子句和
   WHERE 子句合并起来

7. 常量表检测
   首先执行**常量表查询**，然后把查询中涉及到该表的条件全部**替换成常数**，最后再分析其余表的查询成本

   1. 查询的表中一条记录没有，或者只有一条记录（仅适用于MyISAM,Memory）

   2. 使用主键等值匹配或者唯一二级索引列等值匹配作为搜索条件来查询某个表

      ```mysql
      SELECT * FROM table1 INNER JOIN table2
      ON table1.column1 = table2.column2
      WHERE table1.primary_key = 1;
      # 替换后
      SELECT table1表记录的各个字段的常量值, table2.* FROM table1 INNER JOIN table2
      ON table1表column1列的常量值 = table2.column2;
      ```

      



## 外连接消除

凡是不符合WHERE子句中条件的记录都不会参与连接。只要我们在搜索条件中指定关于**被驱动表相关列的值不为 NULL** ，那么外连接中在被驱动表中找不到符合 ON 子句条件的驱动表记录也就被排除出最后的结果集了，也就是说：在这种情况下：**外连接和内连接也就没有什么区别了**

`SELECT * FROM t1 LEFT JOIN t2 ON t1.m1 = t2.m2 WHERE t2.n2 IS NOT NULL`

`SELECT * FROM t1 LEFT JOIN t2 ON t1.m1 = t2.m2 WHERE t2.m2 = 2;`

--> `SELECT * FROM t1 INNER JOIN t2 ON t1.m1 = t2.m2 WHERE t2.m2 = 2;`

在被驱动表的WHERE子句符合空值拒绝的条件后，外连接和内连接可以相互转换。

这种转换带来的好处就是查询优化器可以通过**评估表的不同连接顺序的成本**，选出成本最低的那种连接顺序
来执行查询。



## 子查询优化



### 标量子查询

子查询返回单值

`SELECT * FROM t1 WHERE m1 = (SELECT MIN(m2) FROM t2);`



### 行子查询

子查询返回单行

`SELECT * FROM t1 WHERE (m1, n1) = (SELECT m2, n2 FROM t2 LIMIT 1);`



### 列子查询

子查询返回多行，某列

`SELECT * FROM t1 WHERE m1 IN (SELECT m2 FROM t2);`



### 表子查询

子查询返回某子表，多列

`SELECT * FROM t1 WHERE (m1, n1) IN (SELECT m2, n2 FROM t2);`



### 不相关子查询

子查询不依赖外层表，上面例子皆是不相关



### 相关子查询

子查询依赖外层表

`SELECT * FROM t1 WHERE m1 IN (SELECT m2 FROM t2 WHERE n1 = n2);`



###  IN/  ANY/SOME    /ALL子查询

对于 [NOT] IN/ANY/SOME/AL子查询来说，**子查询中不允许有 LIMIT 语句**

查询某值在不在IN / NOT IN

`SELECT * FROM t1 WHERE (m1, n2) IN (SELECT m2, n2 FROM t2);`

`SELECT * FROM t1 WHERE (m1, n2) NOT IN (SELECT m2, n2 FROM t2);`



ANY/SOME同义，大于小于等于 任何一个或某一个

`SELECT * FROM t1 WHERE m1 > ANY(SELECT m2 FROM t2);`

等价于`SELECT * FROM t1 WHERE m1 > (SELECT MIN(m2) FROM t2)`



AL大于小于或等于所有的值才为`TRUE`

`SELECT * FROM t1 WHERE m1 > ALL(SELECT m2 FROM t2);`

等价于`SELECT * FROM t1 WHERE m1 > (SELECT MAX(m2) FROM t2);`





### 标量/行 子查询执行方式

对于包含不相关的标量子查询或者行子查询的查询语句来说，MySQL会分别独立的执行外层查询和子
查询，就当作两个单表查询就好了



对于相关的标量子查询或者行子查询来说

`SELECT * FROM s1 WHERE key1 = (SELECT common_field FROM s2 WHERE s1.key3 = s2.key3 LIMIT 1);`

1. 先从外层查询中获取一条记录，本例中也就是先从 s1 表中获取一条记录。
2. 然后从上一步骤中获取的那条记录中找出子查询中涉及到的值，本例中就是从 s1 表中获取的那条记录中找
   出 s1.key3 列的值，然后执行子查询。
3. 最后根据子查询的查询结果来检测外层查询 WHERE 子句的条件是否成立，如果成立，就把外层查询的那条记
   录加入到结果集，否则就丢弃。
4. 再次执行第一步，获取第二条外层查询中的记录，依次类推～



### IN子查询优化

**物化表**

不直接**将不相关子查询的结果集**当作外层查询的参数，而是将该结果集写入一个**临时表**里

- 临时表的列就是子查询结果集中的列

- 写入临时表的记录会被去重
- 基于内存的使用 Memory 存储引擎的临时表，而且会为该表建立哈希索引
- 子查询的结果集非常大，超过了系统变量 tmp_table_size 或者 max_heap_table_size ，临时表会转而
  使用基于磁盘的存储引擎来保存结果集中的记录，索引类型也对应转变为 B+ 树索引



**物化表转连接**

子查询变物化表后可以转换为内连接，这样可以通过顺序优化

`SELECT s1.* FROM s1 INNER JOIN materialized_table ON key1 = m_val;`



慢查询分析





# InnoDB的Buffer Pool

提高查询速度，为了缓存磁盘中的页，在 MySQ服务器启动的时候就向操作系统申请了一片连续的内存













# 事务

## 使用

目前只有 `InnoDB` 和 `NDB` 存储引擎支持事务

**开启事务**`BEGIN`

或`START TRANSACTION`可以添加修饰

- `READ ONLY` ：标识当前事务是一个只读事务，也就是属于该事务的数据库操作**只能读取数据**，而不能修
  改数据。
  其实只读事务中只是不允许修改那些其他事务也能访问到的表中的数据，**对于临时表**来说（我们使用CREATE TMEPORARY TABLE创建的表），由于它们只能在当前会话中可见，所以**只读事务其实也是可以对临时表进行增、删、改操作的**。
- `READ WRITE` ：标识当前事务是一个读写事务，也就是属于该事务的数据库操作既可以读取数据，也可
  以修改数据。
- `WITH CONSISTENT SNAPSHOT` ：启动一致性读



**提交事务**`COMMIT`

**手动中止事务**，回滚事务`ROLLBACK`



默认每条语句都自动提交` SHOW VARIABLES LIKE 'autocommit'`

**设置手动提交**`SET autocommit = OFF`



**隐式提交**：即使关闭自动提交，也会自动提交之前的语句

- 定义或修改数据库对象： `CREATE 、ALTER 、 DROP` 等语句去修改**数据库 、 表 、 视图 、 存储过程等**
- 事务控制或关于锁定的语句：
  - 开启一个事务后没结束，但是又开启了第二个事务，会自动提交上一个事务
  - 开始事务后，设置autocommit = ON
  - 加锁` LOCK TABLES 、 UNLOCK TABLES`
-  MySQ复制的一些语句:` START SLAVE 、 STOP SLAVE 、 RESET SLAVE 、 CHANGE MASTER TO`



**保存点**

`SAVEPOINT 保存点名称`

**回滚保存点**`ROLLBACK [WORK] TO [SAVEPOINT] 保存点名称;`

**删除某个保存点** `RELEASE SAVEPOINT 保存点名称`



## 隔离级别

脏写（ Dirty Write ）：如果一个事务**修改**了另一个**未提交事务修改过**的数据

![image-20220427160328029](img\MySQL\image-20220427160328029.png)



脏读（ Dirty Read ）如果一个事务**读到**了另一个**未提交事务修改过**的数据

![image-20220427160352240](img\MySQL\image-20220427160352240.png)



不可重复读（Non-Repeatable Read）
如果一个事务只能读到另一个已经提交的事务修改过的数据，并且**其他事务每对该数据进行一次修改并提交后**，**该事务都能查询得到最新值**

![image-20220427160412601](img\MySQL\image-20220427160412601.png)



幻读（Phantom）
如果一个事务先根据某些条件查询出一些记录，之后另一个事务又向表中插入了符合这些条件的记录，原先的事务再次按照该条件查询时，能把另一个事务插入的记录也读出来

![image-20220427160437481](img\MySQL\image-20220427160437481.png)



严重性：脏写 > 脏读 > 不可重复读 > 幻读

因为脏写太严重了，所以所有的隔离级别都会解决脏写问题。

隔离级别 |脏读| 不可重复读 |幻读
--|--|--|--
READ UNCOMMITTED | × | ×  | × 
READ COMMITTED| √ |  × | × 
REPEATABLE READ | √ | √ | × 
SERIALIZABLE |  √| √ | √



### 具体设置

可以在服务器启动时配置参数` --transaction-isolation=SERIALIZABLE`

`SET [GLOBAL|SESSION] TRANSACTION ISOLATION LEVElevel`

level有四种级别

```mysql
level: {
REPEATABLE READ
| READ COMMITTED
| READ UNCOMMITTED
| SERIALIZABLE
}
```



 `GLOBAL` 关键字（在全局范围影响，已存会话无效）

- 只对执行完该语句之后产生的会话起作用。
- 当前已经存在的会话无效

 `SESSION` 关键字（当前会话范围影响）

- 对当前会话的所有后续的事务有效
- 该语句可以在已经开启的事务中间执行，但不会影响当前正在执行的事务。
- 如果在事务之间执行，则对后续的事务有效。

默认不写时：（下一个用完即恢复）

- 只对当前会话中下一个即将开启的事务有效。
- 下一个事务执行完后，后续事务将恢复到之前的隔离级别。
- 该语句不能在已经开启的事务中间执行，会报错的。



查看`SELECT @@transaction_isolation`





## MVCC原理

通过记录里面隐藏列`trx_id`和`roll_pointer`来记录每个事务的id和回滚链表的头指针，回滚列表通过undo日志组成版本链。

![image-20220427153148656](img\MySQL\image-20220427153148656.png)

版本链的头节点就是当前记录最新的值



这个 `ReadView` 中主要包含4个比较重要的内容：

- `m_ids` ：表示在生成 ReadView 时当前系统中活跃的读写事务的 事务id 列表。

- `min_trx_id` ：表示在生成 ReadView 时当前系统中活跃的读写事务中最小的 事务id ，也就是 m_ids 中的最
  小值。

- `max_trx_id` ：表示生成 ReadView 时系统中应该分配给下一个事务的 id 值。
  小贴士：
  注意max_trx_id并不是m_ids中的最大值，事务id是递增分配的。比方说现在有id为1，2，3这三
  个事务，之后id为3的事务提交了。那么一个新的读事务在生成ReadView时，m_ids就包括1和2，mi
  n_trx_id的值就是1，max_trx_id的值就是4。

- `creator_trx_id` ：表示生成该 ReadView 的事务的 事务id 。
  我们前边说过，只有在对表中的记录做改动时（执行INSERT、DELETE、UPDATE这些语句时）才会为事务分配事务id，否则在一个只读事务中的事务id值都默认为0。

有了这个` ReadView` ，这样在访问某条记录时，只需要按照下边的步骤判断记录的某个版本是否可见：

- 如果**被访问版本的 trx_id 属性值**与 **ReadView 中的 creator_trx_id 值相同**，意味着当前事务在访问它自己
    修改过的记录，所以该版本可以被当前事务访问。
- 如果被访问版本的 trx_id 属性值**小于** ReadView 中的 **min_trx_id 值**，表明生成该版本的事务在当前事务生成 ReadView 前已经提交，所以该版本**可以**被当前事务访问。
- 如果被访问版本的 trx_id 属性值**大于** ReadView 中的 **max_trx_id 值**，表明生成该版本的事务在当前事务生
    成 ReadView 后才开启，所以该版本**不可以**被当前事务访问。
- 如果被访问版本的 trx_id 属性值在 ReadView 的 **min_trx_id 和 max_trx_id 之间**，那就需要判断一下**trx_id 属性值是不是在 m_ids 列表中**，如果**在**，说明创建 ReadView 时生成该版本的事务还是活跃的，该版本**不可以**被访问；如果**不在**，说明创建 ReadView 时生成该版本的事务已经被提交，该版本**可以**被访问。
- 如果**某个版本的数据对当前事务不可见**的话，那就**顺着版本链找到下一个版本**的数据，继续按照上边的步骤判断可见性，依此类推，直到版本链中的最后一个版本。如果最后一个版本也不可见的话，那么就意味着该条记录对该事务完全不可见，查询结果就不包含该记录。



## 解决脏读和不可重复度原理

`READ COMMITTED`隔离级别的事务在**每次查询开始时**都会**生成一个独立的ReadView**，每次提交后再去读时，`ReadView`的`m_ids`列表就去除提交的版本号，那么获取到的数据不会是正在进行的事务，而是已提交的事务，就不会出现脏读。



 `REPEATABLE READ` 隔离级别的事务来说，只会在**第一次执行查询**语句时**生成一个 ReadView** ，之后的查
询就不会重复生成了。那么第一次读取时ReadView和其他事务提交后的ReadView相同，那么再次读取时仍和第一次读取相同。



# 服务器优化

![](C:\java\markdown\docs\img\MySQL\1749651-20190809104145082-1793612311.png)



mysql内存分配需要考虑到操作系统需要使用的内存，其他应用程序所要使用的内存，mysql的会话数以及每个会话使用的内存，然后就是操作系统实例所使用的内存。生产环境的mysql往往都是一个实例独占一个服务，因此，mysql实例需要考虑 mysq的会话数，会话内存以及实例内存。

　　会话内存参数会为每一个连接的会话分配对应大小的内存，相关的主要参数有如下几个：

　　`sort_buffer_size`：会话发送的语句需要进行排序时就会一次性分配对应的大小的缓存

　　`join_buffer_size`：应用程序经常会出现一些两表（或多表）join 的操作需求，Mysq在完成某些 join 需求的时候（al/ index join），为了减少参与 join 的“被驱动表”的读取次数以提高性能，需要使用到 join buffer 来协助完成 join 操作。当 join buffer 太小，mysq不会将该 buffer 存入磁盘文件，而是先将 join buffer 中的结果集与需要 join 的表进行 join 操作，然后清空 join buffer 中的数据，继续将剩余的结果集写入此 buffer 中，如此往复，会造成驱动表需要被多次读取，成倍增加 IO 访问，降低效率。

　　`read_buffer_size`：mysql读入缓冲区的大小。对表进行顺序扫描的请求将分配一个读入缓冲区，mysq会为他分配一段内存缓冲区。read_buffer_size 变量控制这一缓冲区的大小。如果对表的顺序扫描请求非常频繁，并且你认为频繁扫描进行的太慢，可以通过增加该变量以及内存缓冲区大小提高其性能。

　　`read_rnd_buffer_size`：mysq的随机读取缓冲区大小。当按任意顺序读取行时（例如，按照排序顺序），将分配一个随机读缓存区。进行排序查询时，mysq会首先扫描一遍该缓冲，以避免磁盘搜索，提高查询速度，如果需要排序大量数据，可适当调高该值。但 mysq会为每个客户连接发放该缓冲空间，所以应尽量适当设置该值，以避免内存开销过大。

　　`Innodb_buffer_size`：InnoDB 使用该参数指定大小的内存来缓冲数据和索引，这个是 InnoDB引擎中影响性能最大的参数

　　`key_buffer_size`：myisam 决定索引处理的速度，尤其是索引读的速度。默认是 16M，通过检查状态值 key_read_requests（从缓存读取索引的请求次数） 和 key_reads（从磁盘读取索引请求次数），可以知道 key_buffer_size 设置是否合理。比例 key_reads / key_read_requests 应该尽可能的低，至少是 1:100,1:1000 更好（SHOW STATUS LIKE 'key_read%'）。key_buffer_size 只对 MyIsam 表起作用。即使你不使用 MyIsam 表，但是内部的临时磁盘表是 MyIsam 表，也要使用该值。设置该值大小可以通过如下语句获取。



##  InnoDB的变量

查看`show variables like '%innodb%';`

1、innodb_buffer_pool_size
对于InnoDB表来说，innodb_buffer_pool_size的作用就相当于key_buffer_size对于MyISAM表的作用一样。InnoDB使用该参数指定大小的内存来缓冲数据和索引。对于单独的MySQL数据库服务器，最大可以把该值设置成物理内存的80%。
根据MySQL手册，对于2G内存的机器，推荐值是1G（50%）。
show status like 'innodb%';

2、innodb_flush_log_at_trx_commit
主要控制了innodb将log buffer中的数据写入日志文件并flush磁盘的时间点，取值分别为0、1、2三个。0，表示当事务提交时，不做日志写入操作，而是每秒钟将log buffer中的数据写入日志文件并flush磁盘一次；1，则在每秒钟或是每次事物的提交都会引起日志文件写入、flush磁盘的操作，确保了事务的ACID；设置为2，每次事务提交引起写入日志文件的动作，但每秒钟完成一次flush磁盘操作。
实际测试发现，该值对插入数据的速度影响非常大，设置为2时插入10000条记录只需要2秒，设置为0时只需要1秒，而设置为1时则需要229秒。因此，MySQL手册也建议尽量将插入操作合并成一个事务，这样可以大幅提高速度。
根据MySQL手册，在允许丢失最近部分事务的危险的前提下，可以把该值设为0或2。

3、innodb_log_buffer_size
log缓存大小，一般为1-8M，默认为1M，对于较大的事务，可以增大缓存大小。可设置为4M或8M。

4、innodb_additional_mem_pool_size
该参数指定InnoDB用来存储数据字典和其他内部数据结构的内存池大小。缺省值是1M。通常不用太大，只要够用就行，应该与表结构的复杂度有关系。如果不够用，MySQL会在错误日志中写入一条警告信息。
根据MySQL手册，对于2G内存的机器，推荐值是20M，可适当增加。
innodb_thread_concurrency=8
推荐设置为 2*(NumCPUs+NumDisks)，默认一般为8

5、skip-name-resolve
 禁止MySQL对外部连接进行DNS解析，使用这一选项可以消除MySQL进行DNS解析的时间。但需要注意，如果开启该选项，
 则所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求
skip-networking

back_log = 600
 MySQL能有的连接数量。当主要MySQL线程在一个很短时间内得到非常多的连接请求，这就起作用，
 然后主线程花些时间(尽管很短)检查连接并且启动一个新线程。back_log值指出在MySQL暂时停止回答新请求之前的短时间内多少个请求可以被存在堆栈中。
 如果期望在一个短时间内有很多连接，你需要增加它。也就是说，如果MySQL的连接数据达到max_connections时，新来的请求将会被存在堆栈中， 以等待某一连接释放资源，该堆栈的数量即back_log，如果等待连接的数量超过back_log，将不被授予连接资源。
 另外，这值（back_log）限于您的操作系统对到来的TCP/IP连接的侦听队列的大小。
 你的操作系统在这个队列大小上有它自己的限制（可以检查你的OS文档找出这个变量的最大值），试图设定back_log高于你的操作系统的限制将是无效的。

max_connections = 1000
 MySQL的最大连接数，如果服务器的并发连接请求量比较大，建议调高此值，以增加并行连接数量，当然这建立在机器能支撑的情况下，因为如果连接数越多，介于MySQL会为每个连接提供连接缓冲区，就会开销越多的内存，所以要适当调整该值，不能盲目提高设值。可以过’conn%’通配符查看当前状态的连接数量，以定夺该值的大小。

max_connect_errors = 6000
 对于同一主机，如果有超出该参数值个数的中断错误连接，则该主机将被禁止连接。如需对该主机进行解禁，执行：FLUSH HOST。

open_files_limit = 65535
 MySQL打开的文件描述符限制，默认最小1024;当open_files_limit没有被配置的时候，比较max_connections*5和ulimit -n的值，哪个大用哪个，
 当open_file_limit被配置的时候，比较open_files_limit和max_connections*5的值，哪个大用哪个。

table_open_cache = 128
 MySQL每打开一个表，都会读入一些数据到table_open_cache缓存中，当MySQL在这个缓存中找不到相应信息时，才会去磁盘上读取。默认值64
 假定系统有200个并发连接，则需将此参数设置为200*N(N为每个连接所需的文件描述符数目)；
 当把table_open_cache设置为很大时，如果系统处理不了那么多文件描述符，那么就会出现客户端失效，连接不上

max_allowed_packet = 4M
 接受的数据包大小；增加该变量的值十分安全，这是因为仅当需要时才会分配额外内存。例如，仅当你发出长查询或MySQLd必须返回大的结果行时MySQLd才会分配更多内存。
 该变量之所以取较小默认值是一种预防措施，以捕获客户端和服务器之间的错误信息包，并确保不会因偶然使用大的信息包而导致内存溢出。

binlog_cache_size = 1M
 一个事务，在没有提交的时候，产生的日志，记录到Cache中；等到事务提交需要提交的时候，则把日志持久化到磁盘。默认binlog_cache_size大小32K

max_heap_table_size = 8M
 定义了用户可以创建的内存表(memory table)的大小。这个值用来计算内存表的最大行数值。这个变量支持动态改变

tmp_table_size = 16M
 MySQL的heap（堆积）表缓冲大小。所有联合在一个DML指令内完成，并且大多数联合甚至可以不用临时表即可以完成。
 大多数临时表是基于内存的(HEAP)表。具有大的记录长度的临时表 (所有列的长度的和)或包含BLOB列的表存储在硬盘上。
 如果某个内部heap（堆积）表大小超过tmp_table_size，MySQL可以根据需要自动将内存中的heap表改为基于硬盘的MyISAM表。还可以通过设置tmp_table_size选项来增加临时表的大小。也就是说，如果调高该值，MySQL同时将增加heap表的大小，可达到提高联接查询速度的效果

read_buffer_size = 2M
 MySQL读入缓冲区大小。对表进行顺序扫描的请求将分配一个读入缓冲区，MySQL会为它分配一段内存缓冲区。read_buffer_size变量控制这一缓冲区的大小。
 如果对表的顺序扫描请求非常频繁，并且你认为频繁扫描进行得太慢，可以通过增加该变量值以及内存缓冲区大小提高其性能

read_rnd_buffer_size = 8M
 MySQL的随机读缓冲区大小。当按任意顺序读取行时(例如，按照排序顺序)，将分配一个随机读缓存区。进行排序查询时，
 MySQL会首先扫描一遍该缓冲，以避免磁盘搜索，提高查询速度，如果需要排序大量数据，可适当调高该值。但MySQL会为每个客户连接发放该缓冲空间，所以应尽量适当设置该值，以避免内存开销过大

sort_buffer_size = 8M
 MySQL执行排序使用的缓冲大小。如果想要增加ORDER BY的速度，首先看是否可以让MySQL使用索引而不是额外的排序阶段。
 如果不能，可以尝试增加sort_buffer_size变量的大小

join_buffer_size = 8M
 联合查询操作所能使用的缓冲区大小，和sort_buffer_size一样，该参数对应的分配内存也是每连接独享

thread_cache_size = 8
 这个值（默认8）表示可以重新利用保存在缓存中线程的数量，当断开连接时如果缓存中还有空间，那么客户端的线程将被放到缓存中，
 如果线程重新被请求，那么请求将从缓存中读取,如果缓存中是空的或者是新的请求，那么这个线程将被重新创建,如果有很多新的线程，
 增加这个值可以改善系统性能.通过比较Connections和Threads_created状态的变量，可以看到这个变量的作用。(–>表示要调整的值)
 根据物理内存设置规则如下：
 1G —> 8
 2G —> 16
 3G —> 32
 大于3G —> 64

query_cache_size = 8M
MySQL的查询缓冲大小（从4.0.1开始，MySQL提供了查询缓冲机制）使用查询缓冲，MySQL将SELECT语句和查询结果存放在缓冲区中，
 今后对于同样的SELECT语句（区分大小写），将直接从缓冲区中读取结果。根据MySQL用户手册，使用查询缓冲最多可以达到238%的效率。
 通过检查状态值’Qcache_%’，可以知道query_cache_size设置是否合理：如果Qcache_lowmem_prunes的值非常大，则表明经常出现缓冲不够的情况，
 如果Qcache_hits的值也非常大，则表明查询缓冲使用非常频繁，此时需要增加缓冲大小；如果Qcache_hits的值不大，则表明你的查询重复率很低，
 这种情况下使用查询缓冲反而会影响效率，那么可以考虑不用查询缓冲。此外，在SELECT语句中加入SQL_NO_CACHE可以明确表示不使用查询缓冲

query_cache_limit = 2M
指定单个查询能够使用的缓冲区大小，默认1M

key_buffer_size = 4M
指定用于索引的缓冲区大小，增加它可得到更好处理的索引(对所有读和多重写)，到你能负担得起那样多。如果你使它太大，
 系统将开始换页并且真的变慢了。对于内存在4GB左右的服务器该参数可设置为384M或512M。通过检查状态值Key_read_requests和Key_reads，
 可以知道key_buffer_size设置是否合理。比例key_reads/key_read_requests应该尽可能的低，
 至少是1:100，1:1000更好(上述状态值可以使用SHOW STATUS LIKE ‘key_read%’获得)。注意：该参数值设置的过大反而会是服务器整体效率降低

ft_min_word_len = 4
 分词词汇最小长度，默认4

transaction_isolation = REPEATABLE-READ
 MySQL支持4种事务隔离级别，他们分别是：
 READ-UNCOMMITTED, READ-COMMITTED, REPEATABLE-READ, SERIALIZABLE.
 如没有指定，MySQL默认采用的是REPEATABLE-READ，ORACLE默认的是READ-COMMITTED

log_bin = mysql-bin
binlog_format = mixed
expire_logs_days = 30 超过30天的binlog删除
log_error = /data/mysql/mysql-error.log 错误日志路径
slow_query_log = 1
long_query_time = 1 慢查询时间 超过1秒则为慢查询
slow_query_log_file = /data/mysql/mysql-slow.log
performance_schema = 0
explicit_defaults_for_timestamp
lower_case_table_names = 1 不区分大小写
skip-external-locking MySQL选项以避免外部锁定。该选项默认开启
default-storage-engine = InnoDB 默认存储引擎

innodb_file_per_table = 1
 InnoDB为独立表空间模式，每个数据库的每个表都会生成一个数据空间
 独立表空间优点：
 1．每个表都有自已独立的表空间。
 2．每个表的数据和索引都会存在自已的表空间中。
 3．可以实现单表在不同的数据库中移动。
 4．空间可以回收（除drop table操作处，表空不能自已回收）
 缺点：
 单表增加过大，如超过100G
 结论：
 共享表空间在Insert操作上少有优势。其它都没独立表空间表现好。当启用独立表空间时，请合理调整：innodb_open_files

innodb_open_files = 500
 限制Innodb能打开的表的数据，如果库里的表特别多的情况，请增加这个。这个值默认是300

innodb_buffer_pool_size = 64M
 InnoDB使用一个缓冲池来保存索引和原始数据, 不像MyISAM.
 这里你设置越大,你在存取表里面数据时所需要的磁盘I/O越少.
 在一个独立使用的数据库服务器上,你可以设置这个变量到服务器物理内存大小的80%
 不要设置过大,否则,由于物理内存的竞争可能导致操作系统的换页颠簸.
 注意在32位系统上你每个进程可能被限制在 2-3.5G 用户层面内存限制,
 所以不要设置的太高.

innodb_write_io_threads = 4
innodb_read_io_threads = 4

 innodb使用后台线程处理数据页上的读写 I/O(输入输出)请求,根据你的 CPU 核数来更改,默认是4
 注:这两个参数不支持动态改变,需要把该参数加入到my.cnf里，修改完后重启MySQL服务,允许值的范围从 1-64

innodb_thread_concurrency = 0
 默认设置为 0,表示不限制并发数，这里推荐设置为0，更好去发挥CPU多核处理能力，提高并发量

innodb_purge_threads = 1
 InnoDB中的清除操作是一类定期回收无用数据的操作。在之前的几个版本中，清除操作是主线程的一部分，这意味着运行时它可能会堵塞其它的数据库操作。
 从MySQL5.5.X版本开始，该操作运行于独立的线程中,并支持更多的并发数。用户可通过设置innodb_purge_threads配置参数来选择清除操作是否使用单
 独线程,默认情况下参数设置为0(不使用单独线程),设置为 1 时表示使用单独的清除线程。建议为1

innodb_flush_log_at_trx_commit = 2
 0：如果innodb_flush_log_at_trx_commit的值为0,log buffer每秒就会被刷写日志文件到磁盘，提交事务的时候不做任何操作（执行是由mysql的master thread线程来执行的。
 主线程中每秒会将重做日志缓冲写入磁盘的重做日志文件(REDO LOG)中。不论事务是否已经提交）默认的日志文件是ib_logfile0,ib_logfile1
 1：当设为默认值1的时候，每次提交事务的时候，都会将log buffer刷写到日志。
 2：如果设为2,每次提交事务都会写日志，但并不会执行刷的操作。每秒定时会刷到日志文件。要注意的是，并不能保证100%每秒一定都会刷到磁盘，这要取决于进程的调度。
 每次事务提交的时候将数据写入事务日志，而这里的写入仅是调用了文件系统的写入操作，而文件系统是有 缓存的，所以这个写入并不能保证数据已经写入到物理磁盘
 默认值1是为了保证完整的ACID。当然，你可以将这个配置项设为1以外的值来换取更高的性能，但是在系统崩溃的时候，你将会丢失1秒的数据。
 设为0的话，mysqld进程崩溃的时候，就会丢失最后1秒的事务。设为2,只有在操作系统崩溃或者断电的时候才会丢失最后1秒的数据。InnoDB在做恢复的时候会忽略这个值。
 总结
 设为1当然是最安全的，但性能页是最差的（相对其他两个参数而言，但不是不能接受）。如果对数据一致性和完整性要求不高，完全可以设为2，如果只最求性能，例如高并发写的日志服务器，设为0来获得更高性能

innodb_log_buffer_size = 2M
 此参数确定些日志文件所用的内存大小，以M为单位。缓冲区更大能提高性能，但意外的故障将会丢失数据。MySQL开发人员建议设置为1－8M之间

innodb_log_file_size = 32M
 此参数确定数据日志文件的大小，更大的设置可以提高性能，但也会增加恢复故障数据库所需的时间

innodb_log_files_in_group = 3
 为提高性能，MySQL可以以循环方式将日志文件写到多个文件。推荐设置为3

innodb_max_dirty_pages_pct = 90
 innodb主线程刷新缓存池中的数据，使脏数据比例小于90%

innodb_lock_wait_timeout = 120
 InnoDB事务在被回滚之前可以等待一个锁定的超时秒数。InnoDB在它自己的锁定表中自动检测事务死锁并且回滚事务。InnoDB用LOCK TABLES语句注意到锁定设置。默认值是50秒

bulk_insert_buffer_size = 8M
 批量插入缓存大小， 这个参数是针对MyISAM存储引擎来说的。适用于在一次性插入100-1000+条记录时， 提高效率。默认值是8M。可以针对数据量的大小，翻倍增加。

myisam_sort_buffer_size = 8M
 MyISAM设置恢复表之时使用的缓冲区的尺寸，当在REPAIR TABLE或用CREATE INDEX创建索引或ALTER TABLE过程中排序 MyISAM索引分配的缓冲区

myisam_max_sort_file_size = 10G
 如果临时文件会变得超过索引，不要使用快速排序索引方法来创建一个索引。注释：这个参数以字节的形式给出

myisam_repair_threads = 1
 如果该值大于1，在Repair by sorting过程中并行创建MyISAM表索引(每个索引在自己的线程内)

interactive_timeout = 28800
 服务器关闭交互式连接前等待活动的秒数。交互式客户端定义为在mysql_real_connect()中使CLIENT_INTERACTIVE选项的客户端。默认值：28800秒（8小时）

wait_timeout = 28800
 服务器关闭非交互连接之前等待活动的秒数。在线程启动时，根据全局wait_timeout值或全局interactive_timeout值初始化会话wait_timeout值，取决于客户端类型(由mysql_real_connect()的连接选项CLIENT_INTERACTIVE定义)。参数默认值：28800秒（8小时）
 MySQL服务器所支持的最大连接数是有上限的，因为每个连接的建立都会消耗内存，因此我们希望客户端在连接到MySQServer处理完相应的操作后，应该断开连接并释放占用的内存。如果你的MySQServer有大量的闲置连接，他们不仅会白白消耗内存，而且如果连接一直在累加而不断开，最终肯定会达到MySQServer的连接上限数，这会报’too many connections’的错误。对于wait_timeout的值设定，应该根据系统的运行情况来判断。
 在系统运行一段时间后，可以通过show processlist命令查看当前系统的连接状态，如果发现有大量的sleep状态的连接进程，则说明该参数设置的过大，可以进行适当的调整小些。要同时设置interactive_timeout和wait_timeout才会生效。



# mysql主从复制

![img](C:\java\markdown\docs\img\MySQL\16c4d9dd1b8235c3tplv-t2oaga2asx-zoom-in-crop-mark1304000.webp)

步骤一：主库的更新事件(update、insert、delete)被写到binlog

步骤二：从库发起连接，连接到主库。

步骤三：此时主库创建一个binlog dump thread，把binlog的内容发送到从库。

步骤四：从库启动之后，创建一个I/O线程，读取主库传过来的binlog内容并写入到relay log

步骤五：还会创建一个SQL线程，从relay log里面读取内容，从Exec_Master_Log_Pos位置开始执行读取到的更新事件，将更新内容写入到slave的db



## 主从一致的

我们学习数据库的**主从复制原理**后，了解到**从库**拿到并执行主库的binlog日志，就可以保持数据与主库一致了。这是为什么呢？哪些情况会导致不一致呢？

### 长链接

主库和从库在同步数据的过程中断怎么办呢，数据不就会丢失了嘛。因此主库与从库之间维持了一个**长链接**，主库内部有一个线程，专门服务于从库的这个**长链接**的。

### binlog格式

binlog 日志有三种格式，分别是`statement，row和mixed`。

如果是`statement`格式，binlog记录的是**SQL的原文**，如果主库和从库选的索引不一致，可能会导致主库不一致。我们来分析一下。假设主库执行删除这个SQL（其中`a和create_time`都有索引）如下：

```mysql
delete from t where a > '666' and create_time<'2022-03-01' limit 1;
```

我们知道，数据选择了`a`索引和选择`create_time`索引，最后`limit 1`出来的数据一般是不一样的。所以就会存在这种情况：在binlog = `statement`格式时，主库在执行这条SQL时，使用的是索引a，而从库在执行这条SQL时，使用了索引`create_time`。最后主从数据不一致了。

**如何解决这个问题呢？**

可以把binlog格式修改为`row`。`row`格式的`binlog`日志，记录的不是**SQL原文**，而是两个`event:Table_map 和 Delete_rows`。Table_map event说明要操作的表，Delete_rows event用于定义要删除的行为，记录删除的具体行数。`row`格式的binlog记录的就是要删除的主键ID信息，因此不会出现主从不一致的问题。

但是如果SQL删除10万行数据，使用row格式就会很占空间的，10万条数据都在binlog里面，写binlog的时候也很耗IO。但是`statement`格式的binlog可能会导致数据不一致，因此设计MySQL的大叔想了一个折中的方案，`mixed`格式的binlog。所谓的mixed格式其实就是`row`和`statement`格式混合使用，当MySQL判断可能数据不一致时，就用`row`格式，否则使用就用`statement`格式。

## 主从延迟的原因与解决

**主从延迟**是怎么定义的呢？ 与主从数据同步相关的时间点有三个

1. 主库执行完一个事务，写入binlog，我们把这个时刻记为`T1`；
2. 主库同步数据给从库，从库接收完这个binlog的时刻，记录为`T2`；
3. 从库执行完这个事务，这个时刻记录为`T3`。

所谓主从延迟，其实就是指同一个事务，在从库执行完的时间和在主库执行完的时间差值，即`T3-T1`。

**哪些情况会导致主从延迟呢？**

1. 如果从库所在的机器比主库的**机器性能差，会导致主从延迟**，这种情况比较好解决，只需选择**主从库一样规格的机器**就好。
2. 如果**从库的压力大，也会导致主从延迟**。比如主库直接影响业务的，大家可能使用会**比较克制**，因此一般**查询都打到从库**了，结果导致从库查询消耗大量CPU，影响同步速度，最后导致主从延迟。这种情况的话，可以搞了**一主多从**的架构，即多接几个从库分摊读的压力。另外，还可以把binlog接入到Hadoop这类系统，让它们提供查询的能力。
3. **大事务也会导致主从延迟**。如果一个事务执行就要10分钟，那么主库执行完后，给到从库执行，最后这个事务可能就会导致从库延迟10分钟啦。日常开发中，我们为什么特别强调，不要一次性delete太多SQL，需要分批进行，其实也是为了避免大事务。另外，大表的DDL语句，也会导致大事务，大家日常开发关注一下哈。
4. **网络延迟**也会导致主从延迟，这种情况你只能优化你的网络啦，比如带宽20M升级到100M类似意思等。
5. 如果从数据库过多也会导致主从延迟，因此要避免复制的从节点数量过多。从库数据一般以3-5个为宜。
6. 低版本的MySQL只支持单线程复制，如果主库并发高，来不及传送到从库，就会导致延迟。可以换用更高版本的Mysql，可以支持多线程复制。


作者：捡田螺的小男孩
链接：https://juejin.cn/post/7070290856967667742



# 加锁机制

![](C:\java\markdown\docs\img\MySQL\16c52a7ec7a9591atplv-t2oaga2asx-zoom-in-crop-mark1304000.webp)

乐观锁与悲观锁是两种**并发控制**的思想，可用于解决丢失更新问题。

## **乐观锁**

- 每次去取数据，都很乐观，觉得不会出现并发问题。
- 因此，访问、处理数据每次都不上锁。
- 但是在更新的时候，再根据版本号或时间戳判断是否有冲突，有则处理，无则提交事务。

## **悲观锁**

- 每次去取数据，很悲观，都觉得会被别人修改，会有并发问题。
- 因此，访问、处理数据前就加**排他锁**。
- 在整个数据处理过程中锁定数据，事务提交或回滚后才释放锁.

## 锁粒度

- **表锁：** 开销小，加锁快；锁定力度大，发生锁冲突概率高，并发度最低;不会出现死锁。
- **行锁：** 开销大，加锁慢；会出现死锁；锁定粒度小，发生锁冲突的概率低，并发度高。
- **页锁：** 开销和加锁速度介于表锁和行锁之间；会出现死锁；锁定粒度介于表锁和行锁之间，并发度一般

## 兼容性

**共享锁：**

- 又称读锁（S锁）。
- 一个事务获取了共享锁，其他事务可以获取共享锁，不能获取排他锁，其他事务可以进行读操作，不能进行写操作。
- SELECT ... LOCK IN SHARE MODE 显示加共享锁。

**排他锁：**

- 又称写锁（X锁）。
- 如果事务T对数据A加上排他锁后，则其他事务不能再对A加任任何类型的封锁。获准排他锁的事务既能读数据，又能修改数据。
- SELECT ... FOR UPDATE 显示添加排他锁。

## 锁模式

- **记录锁：** 在行相应的索引记录上的锁，锁定一个行记录
- **gap锁：** 是在索引记录间歇上的锁,锁定一个区间
- **next-key锁：** 是记录锁和在此索引记录之前的gap上的锁的结合，锁定行记录+区间。
- **意向锁** 是为了支持多种粒度锁同时存在；



# 问题

## 小数如何存

小数类型为 `decimal`，禁止使用 `float` 和 `double`。
说明：在存储的时候，float 和 double 都存在精度损失的问题，很可能在比较值的时候，得到不正确的
结果。如果存储的数据范围超过 decimal 的范围，建议将数据拆成整数和小数并分开存储。

定点数类型`DECIMAL(M, D)`来说，`M`和`D`都是可选的，`M`代表总长度，`D`代表小数长度，那`M-D`代表整数长度

`M`的范围是`1~65`，`D`的范围是`0~30`，且`D`的值不能超过`M`

```mysql
DECIMAL = DECIMAL(10) = DECIMAL(10, 0) # 默认为整数10位，小数0位
DECIMAL(n) = DECIMAL(n, 0)
```

