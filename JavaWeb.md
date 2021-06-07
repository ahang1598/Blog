[TOC]

# 1. JDBC

## 1.1 基本使用

![image](https://note.youdao.com/yws/public/resource/9a17bf7311b3424a6840aae93c5a8648/xmlnote/9826DF7C19AB4811A1B7B7464EDEE8E9/21292)

**概念**：Java DataBase Connectivity  Java 数据库连接

**JDBC本质**：其实是官方（sun公司）定义的一套操作所有关系型数据库的规则，即接口。各个数据库厂商去实现这套接口，提供数据库驱动jar包。我们可以使用这套接口（JDBC）编程，真正执行的代码是驱动jar包中的实现类。



导入驱动jar包：`mysql-connector-java-5.1.37-bin.jar`

将文件复制到项目包`libs`下，然后右击`libs`  --> Add As Library

类详解：

1. `DriverManager`：驱动管理对象

   1. 注册驱动：告诉程序该使用哪一个数据库驱动jar

      `static void registerDriver(Driver driver)` :注册与给定的驱动程序` DriverManager` 。 
      				写代码使用：  `Class.forName("com.mysql.jdbc.Driver")`;
      				通过查看源码发现：在`com.mysql.jdbc.Driver`类中存在静态代码块

      ```java
          static {
              try {
                  java.sql.DriverManager.registerDriver(new Driver());
              } catch (SQLException E) {
                  throw new RuntimeException("Can't register driver!");
              }
          }
      ```

      *注意：mysql5之后的驱动jar包可以省略注册驱动的步骤。*
      
   2. 获取数据库连接：
       方法：`static Connection getConnection(String url, String user, String password) `
       参数：`url`：指定连接的路径, `user`：用户名`password`：密码 
       语法：`jdbc:mysql://ip地址(域名):端口号/数据库名称`
       例子：`jdbc:mysql://localhost:3306/db3`
       细节：如果连接的是本机`mysql`服务器，并且`mysql`服务默认端口是`3306`，则`url`可以简写为：`jdbc:mysql:///db3`

2. `Connection`：数据库连接对象

   1. 获取执行`sql `的对象
		- `Statement createStatement()`
		- `PreparedStatement prepareStatement(String sql)  `  一般都使用这种方式去定义，更加安全，**防止`SQL`注入**

    2. 管理事务：为了对一套操作完整性的保证
		- 开启事务：`setAutoCommit(boolean autoCommit) `：**调用该方法设置参数为`false`，即开启事务**,默认为`true`，表示开启自动`commit()`，使用位置为执行`SQL`前
		- 提交事务：`commit() `使用位置在执行完所有`SQL`后
		- 回滚事务：`rollback() `使用位置在抓取的异常处，抓一个`Exception`较大的异常类防止因其他异常产生

3. `Statement`：执行`SQL`对象

4. `PreparedStatement`：执行`sql`的对象

   - SQL注入问题：在拼接sql时，有一些sql的特殊关键字参与字符串的拼接。会造成安全性问题	

     输入用户随便，输入密码：a' or 'a' = 'a

     sql：`select * from user where username = 'fhdsjkf' and password = 'a' or 'a' = 'a' `

     为了解决注入问题，在定义SQL语句是通过`?`来对变量占位：`select * from user where username = ? and password = ? `

     然后通过`preparedStatement.setXxx(位置几, 参数)`设置参数` pstmt1.setDouble(1,500);`位置编号从1开始

   - `int executeUpdate(String sql)` ：执行`DML（insert、update、delete`）语句、`DDL(create，alter、drop)`语句
      返回值：影响的行数，可以通过这个影响的行数判断`DML`语句是否执行成功 返回值>0的则执行成功，反之，则失败。

   - `ResultSet executeQuery(String sql)  `：执行`DQL（select)`语句

5. `ResultSet`：查询`SQL`后执行的结果集

    * `boolean next()`: 游标向下移动一行，判断当前行是否是最后一行末尾(是否有数据)，如果是，则返回false，如果不是则返回true
    * `getXxx(参数)`:获取数据
     * `Xxx`：代表数据类型   如： `int getInt() `,	`String getString()`
     * 参数：
    
     	1. int：**代表列的编号,从1开始**   如：` getString(1)`
     	2. String：代表列名称。 如： `getDouble("balance")`

```java
// 基本的通过JDBC接口连接数据库，更新数据库

		Connection con = null;  // 定义在try外部方便后面的finally释放资源
        Statement stam = null;

        try{
            // 1. 注册程序
            Class.forName("com.mysql.jdbc.Driver");
            // 2. 连接数据库
            // 如果本地默认连接可以写成 jdbc:mysql:///test
            con = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "1598");
            // 3. 创建SQL语句
            String s = "update student set sname = '孙七' where sid = 13";
            // 4. 创建SQL对象
            stam = con.createStatement();
            // 5. 执行SQL
            // stam.executeUpdate(s)执行SQL语句后，返回修改的行数，如果大于0则产生了修改
            int count = stam.executeUpdate(s);
            // 6. 处理结果
            if(count > 0) System.out.println("successd!!!");
            else System.out.println("failed!!!");

        }catch (ClassNotFoundException e){
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            // 7. 释放资源，关闭连接
            // 避免空指针异常
            if(stam != null) {
                try {
                    stam.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }

            if(con != null){
                try{
                    con.close();
                }catch(SQLException e){
                    e.printStackTrace();
                }
            }
        }
```



```java
// 对数据库访问用户和年龄，将获取的数据封装成Student类的列表，供后续使用

// 1. 先定义学生类，有姓名和年龄属性
public class Student {
    private String sname;
    private Date sage;
    public String getSname() { return sname; }
    public void setSname(String sname) { this.sname = sname;}
    .....
}

// 2. 定义JDBCDemo3类，一个main方法来使用getEmp()方法返回的学生列表
public static void main(String[] args) {
        List<Student> ls = new JDBCDemo3().getEmp();
        for(Student e: ls){
//            System.out.println(e.getSname() +"---" +e.getSage());
            System.out.println(e);
        }
    }

	// 返回学生类的列表
public List<Student> getEmp(){
        Connection con = null;
        Statement stat = null;
        ResultSet rs = null;
        List<Student> ls = null;
        try {
            Class.forName("com.mysql.jdbc.Driver");
            con = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "1598");
            // stat.executeQuery()查询SQL语句，返回结果集ResultSet，通过next()指针访问行，通过getString("列名称"),或者getString(1)第一列，很特殊第一列不是从0开始而是1。获取该列的字符串，如果该列为Int类型则getInt("列名称")
            String sql = "select * from student where sid = ? ";
            pstat = con.prepareStatement(sql);
            pstat.setString(1, "01");
            rs = pstat.executeQuery(); // 这里面不用再加SQL语句，已经加载了
            Student stu = null;
            ls = new ArrayList<Student>();
            while(rs.next()){
                stu = new Student();
                // 将查询的结果给Student对象，并生成Student的列表，最后返回该列表
                String sname = rs.getString("sname");
                Date sage = rs.getDate("sage");
                stu.setSage(sage);
                stu.setSname(sname);
                ls.add(stu);
            }


        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            try {
                assert rs != null;
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                assert stat != null;
                stat.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                assert con != null;
                con.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        return ls;
    }
```



## 1.2 通过JDBC工具类：JDBCUtils

为了简化书写，自己写一个工具类将一些重复性较强的代码抽取为一个类

通过读取配置文件来方便以后对数据库的一些修改，而且Java代码还不需要变动，只需要修改配置文件即可

在`src`包下新建`jdbc.properties`文件

```java
url=jdbc:mysql://localhost:3306/test
user=root
passwd=root
driver=com.mysql.jdbc.Driver
```

```java
public class JDBCUtils2 {

    private static String url;
    private static String user;
    private static String passwd;
    private static String driver;

    static{
        try {
            Properties pro = new Properties();
            // pro.load(new FileReader("idea_demo\\src\\jdbc.properties"));
            // 自动寻找加载，不会因为后面项目迁移而改动代码
            pro.load( JDBCUtils.class.getClassLoader().getResource("jdbc.properties") );
            url = pro.getProperty("url");
            user = pro.getProperty("user");
            passwd = pro.getProperty("passwd");
            driver = pro.getProperty("driver");
            Class.forName(driver);

        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

    }
	// 连接数据库
    public static Connection getConnect() throws SQLException {
        return DriverManager.getConnection(url, user, passwd);
    }

    // 关闭释放资源
    public static void close(Connection con, PreparedStatement pstat, ResultSet rs){
        if(rs != null){
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if(pstat != null){
            try{
                pstat.close();
            }catch (SQLException e){
                e.printStackTrace();
            }
        }

        if( con != null){
            try{
                con.close();
            }catch(SQLException e){
                e.printStackTrace();
            }
        }

    }
    
    public static void close(Connection con, PreparedStatement pstat){
		close(con, pstat, null); // 当只有更新数据等没有ResultSet返回值时
    }


}
```

```java
        Connection con = null;
        PreparedStatement pstat1 = null;
        PreparedStatement pstat2 = null;
        try {
            con = JDBCUtils2.getConnect();  // 使用自定义的类连接
            // 开启手动提交
            con.setAutoCommit(false);
            String sql1 = "update users set user = ? where id = ? ";
            String sql2 = "update users set user = ? where id = ? ";
            pstat1 = con.prepareStatement(sql1);
            pstat2 = con.prepareStatement(sql2);
            pstat1.setString(1, "zhang");
            pstat1.setInt(2, 1);
            pstat2.setString(1,"ahang");
            pstat2.setInt(2,3);
            int i = pstat1.executeUpdate();
            int s = 3/0;
            int j = pstat2.executeUpdate();
            if(i>0 && j>0) System.out.println("successd");
            else System.out.println("failed");
            // 成功执行后提交
            con.commit();
        } catch (Exception e) {
            e.printStackTrace();
            //事务回滚
            try {
                if(conn != null) {
                    conn.rollback();
                }
        } finally {
            JDBCUtils2.close(con, pstat1, pstat2); //使用自定义的类关闭， 这里可以再JDBCUtils中再加一个close
        }
    }
```



## 1.3 数据库连接池

- 概念：其实就是一个容器(集合)，存放数据库连接的容器。
  		  当系统初始化好后，容器被创建，容器中会申请一些连接对象，当用户来访问数据库时，从容器中获取连接对象，用户访问完之后，会将连接对象归还给容器。

- 优点：节约资源、访问高效

- 实现：

  - 标准接口（自己定义实现）：`DataSource`的`getConnection()`获取连接和`Connection.close()`归还连接而不是关闭连接
  - 由数据库厂商实现：`C3P0`, `Druid`

- C3P0实现

  - 导入jar包：`c3p0-0.9.5.2.jar` 和`mchange-commons-java-0.2.12.jar `，以及数据库驱动MySQL包，并`add as library`

  - 定义配置文件：`c3p0.properties `或者 `c3p0-config.xml`放在`src`下

    ```xml
    <c3p0-config>
      <!-- 使用默认的配置读取连接池对象 -->
      <default-config>
      	<!--  连接参数 -->
        <property name="driverClass">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/test</property>
        <property name="user">root</property>
        <property name="password">root</property>
        
        <!-- 连接池参数 -->
        <!--初始化申请的连接数量-->
        <property name="initialPoolSize">5</property>
        <!--最大的连接数量-->
        <property name="maxPoolSize">10</property>
        <!--超时时间-->
        <property name="checkoutTimeout">3000</property>
      </default-config>
    ```

    

  - ```java
    ComboPooledDataSource ds = new ComboPooledDataSource();  // 资源池连接对象
    for (int i = 0; i < 11; i++) {
        Connection con = ds.getConnection();  // 获取连接
        System.out.println(i + ":" + con);
        if(i == 5) con.close();  // 归还连接
    }
    ```

- Druid实现

  - 导入jar包 `druid-1.0.9.jar`

  - 定义配置文件`druid.properties`

    ```properties
    driverClassName=com.mysql.jdbc.Driver
    url=jdbc:mysql:///test
    username=root
    password=root
    # 初始化连接数量
    initialSize=5
    # 最大连接数
    maxActive=10
    # 最大等待时间
    maxWait=3000
    ```

  - 实现代码

    ```java
    // 先定义一个工具类，供Druid方便使用
    public class DruidUtils {
        private static DataSource ds;
        static{
            Properties pro = new Properties();
            try {
                // 加载配置文件
                pro.load( DruidUtils.class.getClassLoader().getResourceAsStream("druid.properties") );
                // 创建资源池
                ds = DruidDataSourceFactory.createDataSource(pro);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (Exception e) {
                e.printStackTrace();
            }
    
        }
    
        public static Connection getConnect() throws SQLException {
            return ds.getConnection();
        }
    
        // 归还连接
        public static void close(Connection con, PreparedStatement pstat, ResultSet rs){
            if(con != null){
                try {
                    con.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
    
            if(pstat != null){
                try {
                    pstat.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
    
            if(rs != null){
                try {
                    rs.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    
        public static void close(Connection con, PreparedStatement pstat){
            close(con, pstat, null);
        }
    }
    ```

    ```java
    		Connection con = null;
            PreparedStatement pstat = null;
    
            try {
                // 使用自定义的工具类获取连接
                con = DruidUtils.getConnect();
                String sql = "update users set user = 'ahang' where id = 1";
                pstat = con.prepareStatement(sql);
                int count = pstat.executeUpdate();
                if(count > 0) System.out.println("succeed");
            } catch (SQLException e) {
                e.printStackTrace();
            } finally {
                // 使用自定义的工具类归还资源
                DruidUtils.close(con,pstat);
            }
    ```

## 1.4 Spring JDBC

- `Spring`框架对`JDBC`的简单封装。提供了一个`JDBC Template`对象简化JDBC的开发

- 导入`jar`包：`spring-beans-5.0.0.RELEASE.jar`,`spring-core-5.0.0.RELEASE.jar`,`spring-jdbc-5.0.0.RELEASE.jar`,`spring-tx-5.0.0.RELEASE.jar`,`commons-logging-1.2.jar`

- 创建JdbcTemplate对象。依赖于数据源DataSource
  
- `JdbcTemplate template` = `new JdbcTemplate(ds);`
  
- 调用`JdbcTemplate`的方法来完成CRUD的操作
  方法|用途
  --|--
  update()|执行DML语句。增、删、改语句
  queryForMap()|查询结果将结果集封装为map集合，将列名作为key，将值作为value 将这条记录封装为一个map集合，注意：这个方法查询的结果集长度只能是1
  queryForList()|查询结果将结果集封装为list集合，注意：将每一条记录封装为一个Map集合，再将Map集合装载到List集合中
  query()|查询结果，将结果封装为`JavaBean`对象，`query`的参数：`RowMapper`一般我们使用`BeanPropertyRowMapper`实现类。可以完成数据到`JavaBean`的自动封装，`new BeanPropertyRowMapper<类型>(类型.class)`, 对于类型为对象时，如果出现查询的结果有`null`类型数据，在定义对象的变量时使用引用数据类型`Integer`,`Double`而不是`int`,`double`基本数据类型
  queryForObject|查询结果，将结果封装为对象，一般用于聚合函数的查询

```java
public class SpringDemo1 {
    private JdbcTemplate template = new JdbcTemplate(DruidUtils.getDataSource());

    // 使用template.update来更新
    @Test
    public void test1(){
        String sql = "update users set passwd = 'root' where id = 2";
        int count = template.update(sql);
        System.out.println(count);
    }

    // 使用template.update来插入
    @Test
    public void test2(){
        String sql = "insert into users(user, passwd) values(?,?)";
        int count = template.update(sql,"root1", "root");
        System.out.println(count);
    }
	// 使用template.update来删除
    @Test
    public void test3(){
        String sql = "delete from users where id = ?";
        int count = template.update(sql, 4);
        System.out.println(count);
    }
	// 使用template.queryForMap来查询一行数据
    @Test
    public void test4(){
        String sql = "select * from users where id = ?;";
        Map<String, Object> map = template.queryForMap(sql, 3);
        System.out.println(map);  // {user=zhang, passwd=root, id=3}
    }
	// 使用template.queryForList来查询多行数据，每行数据以map存在
    @Test
    public void test5(){
        String sql = "select * from users";
        List<Map<String, Object>> maps = template.queryForList(sql);
        for(Map<String, Object> e: maps){
            System.out.println(e); 
            //{user=ahang, passwd=root, id=1}, {user=root, passwd=root, id=2}
        }
    }
	// 使用template.query来查询多行数据，每行数据以对象形式存在
    @Test
    public void test6(){
        String sql = "select * from student";
        List<Student> list = template.query(sql, new BeanPropertyRowMapper<Student>(Student.class));
        for(Student e: list){
            System.out.println(e);
        }
    }
	// 使用template.queryForObject将结果封装为对象
    @Test
    public void test7(){
        String sql = "select count(*) from student";
        Long count = template.queryForObject(sql, Long.class);
        System.out.println(count);
    }
}
```



# 2. HTML



# 3. CSS



# 4. JavaScript



* 概念：一门客户端脚本语言
	* 运行在客户端浏览器中的。每一个浏览器都有JavaScript的解析引擎
	* 脚本语言：不需要编译，直接就可以被浏览器解析执行了

- 功能：可以来增强用户和html页面的交互过程，可以来控制html元素，让页面有一些动态的效果，增强用户的体验。

`JavaScript` = `ECMAScript` + `JavaScript`自己特有的东西(BOM+DOM)

## 4.1 基本语法

### 4.1.1 使用位置

内联`<script> 语句 </script>`

外联`<script src="myScript.js"></script>`

```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script>alert(a4)</script>  // 可以放在任意位置，但是放在<body>前会先执行脚本后输出HTML页面
</head>
<script>alert(5)</script>
<body>
<script>alert(6)</script>  // 放在<body>后会先输出HTML页面后执行脚本
</body>
<script>alert(7)</script>
</html>
```

### 4.1.2 输出

`alert(abc)`弹出警告框`abc`

`document.write(10)`页面输出文字

```html
<body>

<h2>我的第一张网页</h2>
<p>我的第一个段落。</p>

<p id="demo"></p>  // 在body内通过id定位输出内容， 11
<script>
document.getElementById("demo").innerHTML = 5 + 6;
</script>
</body>
```

### 4.1.3 变量

- 原始数据类型(基本数据类型)：
  - `number`：数字。 整数/小数/
    - `NaN`(`not a number` 一个不是数字的数字类型),用来表示非法的数字，
    - `var x = 100 / "String"`此时`x`表示`NaN`
    - `var x = 100 / "10"`此时`x = 10`，自动转化类型后相除
    - 与`NaN`做运算的结果也为`NaN`：`a = NaN; b = 10; c = 10 + NaN;`
    - 通过`toFixed(i)`精确小数点个数，`x = 9.656; x.toFixed(4);// 返回 9.6560`
  - `string`：字符串。 字符串  "abc" "a" 'abc',用单引号`''`和双引号`""`相同
    - 获取字符串长度`str.length`
    - 特殊字符通过加`\`如：`\'`，`\"`,`\\`
    - 转义字符：`\r`回车，`\t`水平制表符`\v`垂直制表符
    - 当写一行字符串过长产生自动换行，为了方便书写，建议写出`"aaaa   bbb"` 换行 `+ "ccc  ddd"`
  - `boolean`: `true`和`false`
  - `null`：一个对象为空的占位符
  - `undefined`：未定义。如果一个变量没有给初始化值，则会被默认赋值为undefined
- 引用数据类型：对象`var x = {firstName:"Bill", lastName:"Gates"}; `
- 数组:`var cars = ["Porsche", "Volvo", "BMW"];`

- 在开辟变量存储空间时，不定义空间将来的存储数据类型，可以存放任意类型的数据。

  - `var` 变量名 = 初始化值;

  - `typeof`(变量)：获取变量的类型。

### 4.1.4 运算符

- 一元运算符：

  - `++`,`--`自增自减；

  - `+`,`-`正负号。

    - ```javascript
              var b1 = +"10"; // 1. String转化为number,通过给字符前加一个+
              var b2 = +"20";
              var b3 = b1+b2;  // 2. 结果为两个number相加 30
            
            		var bb1 = +"10";
            		var bb2 = "20";
            		var bb3 = bb1 + bb2; // String和number相加结果为"1020"字符串，此时+号表示连接符在两个字符串之间 
            
              var c1 = false;
              var c2 = +true;  // boolean转化为number类型：false = 0, true = 1;
              document.write(c1+c2); // boolean + number = number : 0+1=1结果为 1
      ```

- 算数运算符：`+`,`-`,`*`,`/`

- 赋值运算符：`=`,`+=`,`-=`

- 比较运算符：`>`,`<`,`>=`,`<=`,`==`,`===`全等于。。。

  - 同类型：直接比较，字符串按照字典序逐个字符比较

  - 类型不同：先类型转换后比较

    - ```javascript
              var d1 = "123";
              var d2 = 123;
              document.write(d1 == d2,"<br>");  // 不同类型，先转换同类型后比较相同
              document.write(d1 === d2,"<br>"); // === 全等于，不同类型直接判定为false
      ```

- 逻辑运算符：`&&`,`||`,`!`
  - 其他类型转boolean类型：
    - `number`: `0`或`NaN`为`false`，其余为`true`
    - `string`:除了（"")空字符其余为`true`
    - `null`和`undefined`: `false`
    - 对象：`true`

```javascript
        var f1 = !!"";
        var f2 = !!new Date();
        document.write(f1,"<br>");  // false
        document.write(f2, "<br>");  // true
```

- 三元运算符：`var a =  b > c ? b : c`

  

### 4.1.5 JS不同于Java语法

```javascript
        function f() {
            s3 = 30      // 1. 可以省略结束分号; 2. 在函数内，如果不加var 声明则声明的是全局变量，函数外部也可以使用
            var s4 = 40;  // 函数内的局部变量之能在函数内使用
            document.write(s3);
        }
        f()
        document.write(s3);
        document.write(s4); // 该条语句错误，不能使用函数内局部变量

        switch (a) {  // 这里面的 a 可以为任意类型变量
            case 1: break;
            case "abc":break;
            case true:break;
            case null:break;
            case undefined: break;
            default:break;
        }
```

### 4.1.6 对象

一个拥有属性和方法的变量 ==》 对象

```javascript
        var obj = {
            firstName : "Ahang",  // 属性
            lastName : "Haha",
            fullName : function(){   // 方法
                return this.firstName + this.lastName;
            }
        };
        document.write("<br>", obj.firstName);  // 对象.属性 "Ahang"
        document.write("<br>", obj.fullName());  // 对象.方法() 得到 "AhangHaha"
        document.write("<br>", obj.fullName); // 对象.方法  输出function(){ return this.firstName + this.lastName; }
```

`new`一个对象`var date = new Date()`



### 4.1.7 事件

| 事件        | 描述                         |
| :---------- | :--------------------------- |
| onchange    | HTML 元素已被改变            |
| onclick     | 用户点击了 HTML 元素         |
| onmouseover | 用户把鼠标移动到 HTML 元素上 |
| onmouseout  | 用户把鼠标移开 HTML 元素     |
| onkeydown   | 用户按下键盘按键             |
| onload      | 浏览器已经完成页面加载       |

```javascript
// 通过点击触发事件
<button onclick="document.getElementById('time1').innerHTML = Date()">点击获取时间</button>
<p id="time1"></p>

// 通过点击调用函数
<button onclick="getDate()">点击获取时间</button>
<script>
    function getDate() {
        document.getElementById("time2").innerHTML = Date();
    }
</script>
<p id="time2"></p>

<button onmouseover="this.innerHTML = Date()">移动鼠标到该点上获取时间</button>
```

### 4.1.8 字符串详解



```javascript
var str = "HELLO WORLD";
str[0] = "A";       //可以访问，但不能修改。 不产生错误，但不会工作
str[0];             // 返回 H

// 创建String对象,通过全局函数String()
new String(s);
var str = String(s);

// 转换字符串为数值,通过全局函数parseInt()
var i = parseInt("10");
var f = parseFloat("10.5");
```

**String 对象方法**

| 方法                                                         | 描述                                                 |
| :----------------------------------------------------------- | :--------------------------------------------------- |
| [bold()](https://www.w3school.com.cn/jsref/jsref_bold.asp)   | 使用粗体显示字符串。                                 |
| [charAt()](https://www.w3school.com.cn/jsref/jsref_charAt.asp) | 返回在指定位置的字符。                               |
| [charCodeAt()](https://www.w3school.com.cn/jsref/jsref_charCodeAt.asp) | 返回在指定的位置的字符的 Unicode 编码。              |
| [concat()](https://www.w3school.com.cn/jsref/jsref_concat_string.asp) | 连接字符串。                                         |
| [fontcolor()](https://www.w3school.com.cn/jsref/jsref_fontcolor.asp) | 使用指定的颜色来显示字符串。                         |
| [fontsize()](https://www.w3school.com.cn/jsref/jsref_fontsize.asp) | 使用指定的尺寸来显示字符串。                         |
| [indexOf()](https://www.w3school.com.cn/jsref/jsref_indexOf.asp) | 检索字符串。                                         |
| [italics()](https://www.w3school.com.cn/jsref/jsref_italics.asp) | 使用斜体显示字符串。                                 |
| [lastIndexOf()](https://www.w3school.com.cn/jsref/jsref_lastIndexOf.asp) | 从后向前搜索字符串。                                 |
| [link()](https://www.w3school.com.cn/jsref/jsref_link.asp)   | 将字符串显示为链接。                                 |
| [localeCompare()](https://www.w3school.com.cn/jsref/jsref_localeCompare.asp) | 用本地特定的顺序来比较两个字符串。                   |
| [match()](https://www.w3school.com.cn/jsref/jsref_match.asp) | 找到一个或多个正则表达式的匹配。                     |
| [replace()](https://www.w3school.com.cn/jsref/jsref_replace.asp) | 替换与正则表达式匹配的子串。                         |
| [search()](https://www.w3school.com.cn/jsref/jsref_search.asp) | 检索与正则表达式相匹配的值。                         |
| [slice()](https://www.w3school.com.cn/jsref/jsref_slice_string.asp) | 提取字符串的片断，并在新的字符串中返回被提取的部分。 |
| [split()](https://www.w3school.com.cn/jsref/jsref_split.asp) | 把字符串分割为字符串数组。                           |
| [strike()](https://www.w3school.com.cn/jsref/jsref_strike.asp) | 使用删除线来显示字符串。                             |
| [sub()](https://www.w3school.com.cn/jsref/jsref_sub.asp)     | 把字符串显示为下标。                                 |
| [substr()](https://www.w3school.com.cn/jsref/jsref_substr.asp) | 从起始索引号提取字符串中指定数目的字符。             |
| [substring()](https://www.w3school.com.cn/jsref/jsref_substring.asp) | 提取字符串中两个指定的索引号之间的字符。             |
| [sup()](https://www.w3school.com.cn/jsref/jsref_sup.asp)     | 把字符串显示为上标。                                 |
| [toLowerCase()](https://www.w3school.com.cn/jsref/jsref_toLowerCase.asp) | 把字符串转换为小写。                                 |
| [toUpperCase()](https://www.w3school.com.cn/jsref/jsref_toUpperCase.asp) | 把字符串转换为大写。                                 |
| [toString()](https://www.w3school.com.cn/jsref/jsref_toString_string.asp) | 返回字符串。                                         |
| [valueOf()](https://www.w3school.com.cn/jsref/jsref_valueOf_string.asp) | 返回某个字符串对象的原始值。                         |

### 4.1.9 数组详解

```javascript
        var arr = ["10", 20, true, null, f(), "abc"];   // 可以存储多个不同类型的变量及函数
        for(var i = 0; i < arr.length; i++){
            document.write("<br>")
            document.write(arr[i]);
            document.write(" : " + typeof(arr[i]));
        }
```

数组长度：`arr.length`

**Array 对象方法**

| 方法                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [concat()](https://www.w3school.com.cn/jsref/jsref_concat_array.asp) | 连接两个或更多的数组，并返回结果。`arr1.concat(arr2, arr3,...);` |
| [join()](https://www.w3school.com.cn/jsref/jsref_join.asp)   | 把数组的所有元素放入一个字符串。元素通过指定的分隔符进行分隔。 |
| [pop()](https://www.w3school.com.cn/jsref/jsref_pop.asp)     | 删除并返回数组的最后一个元素`fruits.pop();`                  |
| [push()](https://www.w3school.com.cn/jsref/jsref_push.asp)   | 向数组的**末尾添加**一个或更多元素，并返回新的长度。`fruits.push("Kiwi");` |
| [reverse()](https://www.w3school.com.cn/jsref/jsref_reverse.asp) | 颠倒数组中元素的顺序。可以先`fruits.sort();`后`fruits.reverse();`实现降序 |
| [shift()](https://www.w3school.com.cn/jsref/jsref_shift.asp) | 与`pop`对应，删除并返回数组的第一个元素`fruits.shift();`     |
| [slice()](https://www.w3school.com.cn/jsref/jsref_slice_array.asp) | 从某个已有的数组返回选定的元素,`fruits.slice(1);`从第二个元素开始到最后的数组；`fruits.slice(1, 3);`返回从下标为1到下标为3（不包含3）的数组 |
| [sort()](https://www.w3school.com.cn/jsref/jsref_sort.asp)   | 对数组的元素进行排序`fruits.sort();`，里面可以放置函数以操控排序：`points.sort(function(a, b){return b - a});` |
| [splice()](https://www.w3school.com.cn/jsref/jsref_splice.asp) | 删除元素，并向数组添加新元素。`var fruits = ["Banana", "Orange", "Apple", "Mango"]; fruits.splice(2, 0, "Lemon", "Kiwi");`第一个参数（2）定义了应添加新元素的位置（拼接）。第二个参数（0）定义应删除多少元素。其余参数（“Lemon”，“Kiwi”）定义要添加的新元素。返回的数组为`Banana,Orange,Lemon,Kiwi,Apple,Mango` |
| [toString()](https://www.w3school.com.cn/jsref/jsref_toString_array.asp) | 把数组转换为字符串，并返回结果。`fruits.toString();`返回`"a,b,c,d"`以逗号分隔的字符串 |
| [toLocaleString()](https://www.w3school.com.cn/jsref/jsref_toLocaleString_array.asp) | 把数组转换为本地数组，并返回结果。                           |
| [unshift()](https://www.w3school.com.cn/jsref/jsref_unshift.asp) | 与`push`对应，向数组的**开头添加**一个或更多元素，并返回新的长度。`fruits.unshift("Lemon");` |
| [valueOf()](https://www.w3school.com.cn/jsref/jsref_valueof_array.asp) | 返回数组对象的原始值                                         |

### 4.1.10 数学

使用方式：`Math.abs(-10)`

**Math 对象方法**

| 方法             | 描述                                                     |
| :--------------- | :------------------------------------------------------- |
| abs(x)           | 返回 x 的绝对值                                          |
| acos(x)          | 返回 x 的反余弦值，以弧度计                              |
| asin(x)          | 返回 x 的反正弦值，以弧度计                              |
| atan(x)          | 以介于 -PI/2 与 PI/2 弧度之间的数值来返回 x 的反正切值。 |
| atan2(y,x)       | 返回从 x 轴到点 (x,y) 的角度                             |
| ceil(x)          | 对 x 进行上舍入                                          |
| cos(x)           | 返回 x 的余弦                                            |
| exp(x)           | 返回 Ex 的值                                             |
| floor(x)         | 对 x 进行下舍入                                          |
| log(x)           | 返回 x 的自然对数（底为e）                               |
| max(x,y,z,...,n) | 返回最高值                                               |
| min(x,y,z,...,n) | 返回最低值                                               |
| pow(x,y)         | 返回 x 的 y 次幂                                         |
| random()         | 返回 0 ~ 1 之间的随机数                                  |
| round(x)         | 把 x 四舍五入为最接近的整数                              |
| sin(x)           | 返回 x（x 以角度计）的正弦                               |
| sqrt(x)          | 返回 x 的平方根                                          |
| tan(x)           | 返回角的正切                                             |

### 4.1.11 正则表达式

语法： `/表达式/修饰符` ==》 `/haha/i`对`haha`匹配，且由于修饰符为`i`对大小写忽略

使用：`str.search(/w3school/i); `,`str.replace(/microsoft/i, "W3School"); `

| 修饰符 | 描述                                                     |
| :----- | :------------------------------------------------------- |
| i      | 执行对大小写不敏感的匹配。                               |
| g      | 执行全局匹配（查找所有匹配而非在找到第一个匹配后停止）。 |
| m      | 执行多行匹配。                                           |

**RegExp 对象方法**

| 方法                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [compile](https://www.w3school.com.cn/jsref/jsref_regexp_compile.asp) | 编译正则表达式。                                             |
| [exec](https://www.w3school.com.cn/jsref/jsref_exec_regexp.asp) | 检索字符串中指定的值。返回找到的值，并确定其位置。`/e/.exec("The best things in life are free!");` |
| [test](https://www.w3school.com.cn/jsref/jsref_test_regexp.asp) | 检索字符串中指定的值。返回 true 或 false。`/e/.test("The best things in life are free!");` |

支持正则表达式的 String 对象的方法

| 方法                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [search](https://www.w3school.com.cn/jsref/jsref_search.asp) | 检索与正则表达式相匹配的值。`str.search(regexp)`             |
| [match](https://www.w3school.com.cn/jsref/jsref_match.asp)   | 找到一个或多个正则表达式的匹配。`str.match(regexp)`          |
| [replace](https://www.w3school.com.cn/jsref/jsref_replace.asp) | 替换与正则表达式匹配的子串。`str.match(regexp,string)`或者替换的字符串可以通过已匹配的字符串相比较`$1...` |
| [split](https://www.w3school.com.cn/jsref/jsref_split.asp)   | 把字符串分割为字符串数组。`str.split(regexp)`或者加上只显示前几个字符：`str.split(regexp, 3)` |





# 5. MAVEN安装使用

## 5.1 安装

1. 下载安装包后解压缩至无中文路径的文件夹中
2. 设置系统变量：`MAVEN_HOME`为`D:\develop\apache-maven-3.5.2`
3. 设置`PATH`添加`%MAVEN_HOME%\bin`
4. 通过`cmd`输入`mvn -v`测试



## 5.2 使用

[具体使用教程]: https://www.bilibili.com/video/BV12J411M7Sj?p=5

- 对于第一次使用创建项目，然后在项目里面创建多个`module`模块
- 不同的模块下`pom.xml`之间默认相互独立，项目下的`pom.xml`和模块下的`pom.xml`默认时相互独立的，也就是说子模块下的依赖包和主项目下的依赖包默认相互独立，不共享
- 子模块在创建时IDE中选择继承自主项目，则子模块下的会显示有主项目的jar包；也可以选择继承另一个子模块，则会有另一个子模块下的jar包
- 子模块可以自主选择继承的父模块

```xml
    <!--此处设置父模块的信息：id和组id-->
	<parent>
        <artifactId>MyBatis</artifactId>
        <groupId>MyBatis</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
	<!--此处设置本模块的Id-->
    <artifactId>MyBatis_3</artifactId>
```



新建项目后--新建模块---为了后面的`JavaWeb`，使用现成的`webapp`去建立

![0](https://note.youdao.com/yws/public/resource/30e1580fd8278c166df6e1fe1d9f4cf1/xmlnote/6F28DDA263A04809821841590BB7D810/2084)



## 5.3 Tomcat安装使用

下载：http://tomcat.apache.org/

安装：直接解压缩文件即可

使用：通过`bin\startup.bat`，浏览器通过`http://localhost:8080`即可访问，关闭：`bin/shutdown.bat`

部署项目的方式：

   			1. 直接将项目放到`webapps`目录下即可。
   	        * /hello：项目的访问路径-->虚拟目录
   	            * 简化部署：将项目打成一个war包，再将war包放置到webapps目录下。
   	                * war包会自动解压缩

  2. 配置conf/server.xml文件,在<Host>标签体中配置
     <Context docBase="D:\hello" path="/hehe" />`docBase:项目存放的路径

     path：虚拟目录

3. **推荐：在conf\Catalina\localhost创建任意名称的xml文件。**

   在文件中编写`<Context docBase="D:\hello" />`

   虚拟目录：xml文件的名称

## 5.4 配置Tomcat

新建一个maven的module后配置TomCat服务器

![第一步](https://note.youdao.com/yws/public/resource/30e1580fd8278c166df6e1fe1d9f4cf1/xmlnote/340EA7631C604763B805C1BDA449D181/2069)

![2](https://note.youdao.com/yws/public/resource/30e1580fd8278c166df6e1fe1d9f4cf1/xmlnote/6AAC4C2258374220BE273783ADF8B071/2065)

![3](https://note.youdao.com/yws/public/resource/30e1580fd8278c166df6e1fe1d9f4cf1/xmlnote/CFE07325C7E54080814F15DD0152E30D/2064)

![4](https://note.youdao.com/yws/public/resource/30e1580fd8278c166df6e1fe1d9f4cf1/xmlnote/16F5FEE6AF7D4944BD25212DC744E4ED/2066)

![6](https://note.youdao.com/yws/public/resource/30e1580fd8278c166df6e1fe1d9f4cf1/xmlnote/BEE58E22D5F8421587DC535373B255F7/2088)

如果没有war包需要手动打包一个教程：[手动war包](https://blog.csdn.net/lamfang/article/details/84723678)



  2. 

# 6. Servlet使用

## 6.1 基本概念

`Servlet`就是一个接口，定义了Java类被浏览器访问到(`tomcat`识别)的规则。

定义一个类实现`servlet`接口：`public class ServletDemo1 implements Servlet`

然后在`web.xml`中配置：

```xml
	    <!--配置Servlet -->
	    <servlet>
	        <servlet-name>demo1</servlet-name>
	        <servlet-class>cn.itcast.web.servlet.ServletDemo1</servlet-class>
	    </servlet>
	
	    <servlet-mapping>
	        <servlet-name>demo1</servlet-name>
	        <url-pattern>/demo1</url-pattern>
	    </servlet-mapping>
```

**执行原理**

1. 当服务器接受到客户端浏览器的请求后，会解析请求`URL`路径，获取访问的`Servlet`的资源路径

2. 查找`web.xml`文件，是否有对应的`<url-pattern>`标签体内容。

3. 如果有，则在找到对应的`<servlet-class>`全类名

4. `tomcat`会将字节码文件加载进内存，并且创建其对象

5. 调用其方法

   

   **在`Servlet 3.0`中无需`web.xml`，通过注解配置**

     1. 创建JavaEE项目，选择Servlet的版本3.0以上，可以不创建web.xml
     2. 定义一个类，实现Servlet接口
     3. 复写方法
     4. 在类上使用`@WebServlet`注解，进行配置
        * `@WebServlet`("资源路径")

**Servlet中的生命周期方法：**

> 1. 被创建：执行`init`方法，只执行一次
>    **`Servlet`什么时候被创建？**
>    默认情况下，第一次被访问时，`Servlet`被创建
>    可以配置执行`Servlet`的创建时机。
>    **在`<servlet>`标签下配置**
>    1. 第一次被访问时，创建`<load-on-startup>`的值为负数
>    2. 在服务器启动时，创建`<load-on-startup>`的值为0或正整数
>       `Servlet`的`init`方法，只执行一次，说明一个`Servlet`在内存中只存在一个对象，`Servlet`是单例的
>       多个用户同时访问时，**可能存在线程安全问题**。
>       解决：尽量不要在`Servlet`中定义成员变量。即使定义了成员变量，也不要对修改值
> 2. 提供服务：执行`service`方法，执行多次
>    每次访问`Servlet`时，`Service`方法都会被调用一次。
> 3. 被销毁：执行`destroy`方法，只执行一次
>    `Servlet`被销毁时执行。服务器关闭时，`Servlet`被销毁
>    只有服务器正常关闭时，才会执行`destroy`方法。
>    `destroy`方法在`Servlet`被销毁之前执行，一般用于释放资源

```java
// 在项目的src下创建Java文件，

@WebServlet("/demo") 
// 通过WebServlet("资源路径")注解配置，给该Java资源起的名称，也叫做网页访问的资源路径，在web上通过http://localhost:8080/demo访问该资源
public class servlet_demo1 implements Servlet {

    @Override // 初始化，默认在第一次访问时使用
    public void init(ServletConfig servletConfig) throws ServletException {System.out.println("init...");}
    @Override
    public ServletConfig getServletConfig() {return null;}

    @Override  // 每次web调用该服务，便执行该方法
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("service...");
    }
    @Override
    public String getServletInfo() {return null;}
    @Override  // 最后正常结束时销毁
    public void destroy() {System.out.println("destroy...");}
}
```

## 6.2 **`maven`中使用`servlet`具体使用步骤：**

### 6.2.1 构建Maven项目

删掉里面的src目录，以后我们的学习就在这个项目里面建立Moudel；这个空的工程就是Maven主工程；

![image-20210124181616816](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210124181616.png)



**关于Maven父子工程的理解：**

父项目中会有

```java
<modules>
    <module>servlet-01</module>
</modules>

```

子项目会有   (local history ---->accept)

```java
<parent>
    <artifactId>javaweb-02-servlet</artifactId>
    <groupId>com.kuang</groupId>
    <version>1.0-SNAPSHOT</version>
</parent>

```

父项目中的`jar`包子项目可以直接使用



### 6.2.2 Maven环境优化

-  修改`web.xml`为最新的

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                             http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0"
         metadata-complete="true">

</web-app>
```

- ​	在`maven`的`pom`文件中添加依赖，为了后面使用`jsp`和`servlet`作准备

  ```xml
  <dependencies>
          <dependency>
              <groupId>javax.servlet</groupId>
              <artifactId>javax.servlet-api</artifactId>
              <version>4.0.1</version>
  
          </dependency>
  
          <dependency>
              <groupId>javax.servlet.jsp</groupId>
              <artifactId>jsp-api</artifactId>
              <version>2.0</version>
          </dependency>
      </dependencies>
  ```

- 在maven的pom中添加资源路径，防止导入资源失败

```xml
	<build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
```



-  将maven的结构搭建完整

![image-20210124185202442](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210124185202.png)

### 6.2.3 编写一个Servlet程序

> 编写一个普通类 HelloServlet
>
> 实现Servlet接口，这里我们直接继承HttpServlet

![image-20210124185325781](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210124185325.png)

==重写方法（ctrl+o 快捷键）==

```java
package com.kuang.servlet;

public class HelloServlet extends HttpServlet {

    //由于get或者post只是请求实现的不同的方式，可以相互调用，业务逻辑都一样；
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //ServletOutputStream outputStream = resp.getOutputStream();
        PrintWriter writer = resp.getWriter(); //响应流
        writer.print("Hello,Serlvet");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}

```

### 6.2.4 编写Servlet的映射

为什么需要映射：我们写的是JAVA程序，但是要通过浏览器访问，而浏览器需要连接web服务器(**web.xml**)，所以我们需要再web服务中注册我们写的Servlet，还需给他一个浏览器能够访问的路径；

```java
	<!--注册Servlet-->
    <servlet>
        <servlet-name>hello</servlet-name>
        <servlet-class>com.ylw.servlet.HelloServlet</servlet-class>
    </servlet>
    <!--Servlet的请求路径，上面的name和下面的name要一样-->
    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
```

### 6.2.5 配置Tomcat

详见 5.4

### 6.2.6 启动测试

详见6.4

![image-20210124190418905](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210124190419.png)



## 6.3 web.xml的配置mapper问题

1. 一个Servlet可以指定一个映射路径

```xml
<servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
</servlet-mapping>
```

2.一个Servlet可以指定多个映射路径

```xml
    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello1</url-pattern>
    </servlet-mapping>
```

3. 一个Servlet可以指定通用映射路径

```xml
   <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello/*</url-pattern>
    </servlet-mapping>

```

4. 默认请求路径

```xml
  <!--默认请求路径-->
    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>

```

5. 指定一些后缀或者前缀等等….注意：这时候 * 前面就不要加路径了

```xml
<!--可以自定义后缀实现请求映射
    注意点，*前面不能加项目映射的路径
    hello/sajdlkajda.qinjiang
    -->
<servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>*.do</url-pattern>
</servlet-mapping>
```

6. 优先级问题 指定了固有的映射路径优先级最高，如果找不到就会走默认的(/*)处理请求；

```xml
<!--404-->
<servlet>
    <servlet-name>error</servlet-name>
    <servlet-class>com.kuang.servlet.ErrorServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>error</servlet-name>
    <url-pattern>/*</url-pattern>
</servlet-mapping>
```

7. 如果默认(/\*)和前后缀的(*.do)同时存在时，会使用默认的

## 6.4 TomCat服务重启热加载

- Tomcat服务重启 -----> 加载新添加的`Java`文件和资源文件`properties`

- 如果仅仅对Java文件进行了修改，而没有添加新的Java，可以通过热加载不重启服务

![0](https://note.youdao.com/yws/public/resource/30e1580fd8278c166df6e1fe1d9f4cf1/xmlnote/7C3C6267A81C47C3AF6D3A9CC0CA54AF/2090)

​																									首先使用debug

![](https://note.youdao.com/yws/public/resource/30e1580fd8278c166df6e1fe1d9f4cf1/xmlnote/F50B4D7F44304E9D96CDE38F72BFFAD5/2092)

- 如果只对`web.xml`配置文件进行了修改，可以通过`redeploy`来更新

![](https://note.youdao.com/yws/public/resource/30e1580fd8278c166df6e1fe1d9f4cf1/xmlnote/F51604B623404E9A979B37ED7835606F/2093)

- 如果同时添加了新文件`Java`和修改了`web.xml`的话，需要重启服务，然后`redeploy`



## 6.5 **ServletContext 常用方法**

web容器在启动的时候，它会为每个`web`程序都创建一个对应的`ServletContext对象`，它代表了当前的web应用；
主要方法：

```java
ServletContext context = this.getServletContext();
```

### 1、共享数据

用到的主要方法：

```java
//设置
context.setAttribute("参数名",参数值（可以是变量名）);
//获取,如果没有先set直接get会创建一个，并赋值为null
context.getAttribute("参数名");
```

在这个Servlet中保存的数据，可以在另外一个servlet中拿到；
![在这里插入图片描述](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210124200101.png)
在`HelloServlet` 里设置上下文数据

```java
package com.kuang.servlet;

public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        //this.getInitParameter() 初始化参数
        //this.getServletConfig() Servlet配置
        //this.getServletContext() Servlet上下文
        ServletContext context = this.getServletContext();

        String username = "ahang"; //数据
        context.setAttribute("username",username); //将一个数据保存在了ServletContext，数据的名字为username，值为变量username的值

        System.out.println("Hello,进入doGet");
    }
}

```

在`GetServlet `里获取上下文

```java
package com.kuang.servlet;

public class GetServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext context = this.getServletContext(); //这里获取的上下文就是HellServlet里设置的

        String username = (String) context.getAttribute("username"); //获取上下文中名为username的值

        resp.setContentType("text/html"); //设置文本类型
        resp.setCharacterEncoding("utf-8"); //设置编码
        resp.getWriter().print("用户名："+username); //写在网页上
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp); //注意这里改成doGet(req, resp)，形成一个规范
    }
}

```

web.xml里的注册

```xml
 <servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.kuang.servlet.HelloServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>

  <servlet>
    <servlet-name>getContext</servlet-name>
    <servlet-class>com.kuang.servlet.GetServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>getContext</servlet-name>
    <url-pattern>/getContext</url-pattern>
  </servlet-mapping>

```

重启Tomcat。
然后打开浏览器，先访问/hello进行设置，再访问/getContext进行获取
![image-20210124200936267](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210124200936.png)

![image-20210124200925027](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210124200925.png)

### 2、获取初始化参数

用到的主要方法：

```java
context.getInitParameter("参数名");
```

先在`web.xml`文件里配置初始参数

```bash
    <!--配置一些web应用的初始化参数-->
    <context-param>
        <param-name>url</param-name>
        <param-value>jdbc:mysql://localhost:3306/mybatis</param-value>
    </context-param>
```

写一个类获取

```java
package com.kuang.servlet;

public class ServletDemo03 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext context = this.getServletContext(); //这里获取的上下文就是HellServlet里设置的
        String url = context.getInitParameter("url"); //获取上web.xml文件中设置的参数<context-param>

        resp.getWriter().print(url); //写在网页上
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);//注意这里改成doGet(req, resp)，形成一个规范
    }
}


```

在web.xml里注册这个类

```bash
<servlet>
    <servlet-name>gp</servlet-name>
    <servlet-class>com.kuang.servlet.ServletDemo03</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>gp</servlet-name>
    <url-pattern>/gp</url-pattern>
  </servlet-mapping>
```

重启Tomcat。
然后打开浏览器，访问/gp进行获取
![image-20210124232352690](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210124232352.png)

### 3、请求转发

A要访问C中的资源：有两种方式

![image-20210125001115628](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210125001115.png)

用到的主要方法：

```java
context.getRequestDispatcher("转发路径").forward(req,resp);
```

转发代码

```java
package com.kuang.servlet;

public class ServletDemo04 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext context = this.getServletContext(); //这里获取的上下文就是HellServlet里设置的
        System.out.println("进入了Demo04");
//        RequestDispatcher requestDispatcher = context.getRequestDispatcher("/gp"); //设置转发路径
//        requestDispatcher.forward(req,resp); //调用forward实现请求转发
        //合并上面两句
        context.getRequestDispatcher("/gp").forward(req,resp);

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);//注意这里改成doGet(req, resp)，形成一个规范
    }
}

```

在web.xml里注册类

```xml
    <servlet>
        <servlet-name>sd4</servlet-name>
        <servlet-class>com.ylw.servlet.ServletDemo04</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>sd4</servlet-name>
        <url-pattern>/sd4</url-pattern>
    </servlet-mapping>

```

重启Tomcat，
打开浏览器，进入/sd4，发现路径没有变，但是显示的是/gp的页面，这就是转发的作用（重定向才会改变路径）
![image-20210125001442817](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210125001442.png)
由于我们设置了输出语句，在控制台里可以看到，确实进入了demo04，只不过显示的是转发页面
![image-20210125001453657](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210125001453.png)

### 4、读取资源文件 getResourceAsStream()

用到的主要方法：

```java
//获取文件，变成流
this.getServletContext().getResourceAsStream("文件路径"）
//加载流
Properties prop = new Properties();
prop.load(inputStream); //加载上面文件转化成的流
prop.getProperty("属性名"); //获取文件中的一个属性

```

Properties

比较分析：

- 在java目录下新建properties
- 在resources目录下新建properties

发现：都被打包到了同一个路径下：classes，我们俗称这个路径为classpath:

![image-20210125001958291](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210125001958.png)

在java目录下的properties，发现导出的target中没有该文件，就需要在`pom.xml`中添加配置，详见博客https://blog.csdn.net/qq_43594119/article/details/106199248中的**解决资源导出失败问题**

读取资源文件：
1在resources目录下新建一个db.properties文件作为资源文件
![image-20210125001727604](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210125001727.png)

```properties
username=wuxinyu
password=123456
```

2.写一个类用来读取这个资源文件，下面是截图和代码
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521202347385.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNTk0MTE5,size_16,color_FFFFFF,t_70)

```java
package com.kuang.servlet;

public class PropertiesServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        //获得资源，变成一个流  路径是target下面的路径
        InputStream inputStream = this.getServletContext().getResourceAsStream("/WEB-INF/classes/db.properties");

        Properties prop = new Properties();
        prop.load(inputStream); //加载上面文件转化成的流
        String user = prop.getProperty("username"); //获取文件中的一个属性
        String pwd = prop.getProperty("password"); //获取文件中的一个属性

        //在网页上显示出来
        resp.getWriter().print("user: " + user +'\n' + "pwd: " + pwd);

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp); //注意这里改成doGet(req, resp)，形成一个规范
    }
}


```

注意这里的路径是 `/WEB-INF/classes/db.properties` 第一个 / 代表当前项目
![在这里插入图片描述](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210125001931.png)
3.在web.xml中注册该类

```bash
    <servlet>
        <servlet-name>sd5</servlet-name>
        <servlet-class>com.ylw.servlet.ServletDemo05</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>sd5</servlet-name>
        <url-pattern>/sd5</url-pattern>
    </servlet-mapping>
```

4.重启tomcat，访问/sd5
![image-20210125002355108](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210125002355.png)

## 6.6、HttpServletResponse

web服务器接收到客户端的http请求，针对这个请求，分别创建一个代表请求的HttpServletRequest对象，代表响应的一个HttpServletResponse；



- 如果要获取客户端请求过来的参数：找HttpServletRequest
- 如果要给客户端响应一些信息：找HttpServletResponse

### 1、简单分类

负责向浏览器发送数据的方法

```java
 servletOutputstream getOutputstream() throws IOException;
 Printwriter getwriter() throws IOException;
```

负责向浏览器发送响应头的方法

### 2、下载文件

1. 向浏览器输出消息（一直在讲，就不说了）
2. 下载文件
   1. 要获取下载文件的路径
   2. 下载的文件名是啥？
   3. 设置想办法让浏览器能够支持下载我们需要的东西
   4. 获取下载文件的输入流
   5. 创建缓冲区
   6. 获取OutputStream对象
   7. 将FileOutputStream流写入到bufer缓冲区
   8. 使用OutputStream将缓冲区中的数据输出到客户端！

```java
package com.kuang.servlet;

public class FileServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
//        1. 要获取下载文件的路径
        String realPath = "D:\\WorkingSpace\\Idea_workingspace\\JavaWeb\\javaweb-02-servlet\\response\\src\\main\\resources\\1.png";
        System.out.println("下载文件的路径为："+realPath);
//        2. 下载的文件名是啥？
        String fileName = realPath.substring(realPath.lastIndexOf("\\") + 1);
//        3. 设置想办法让浏览器能够支持下载我们需要的东西
        System.out.println(fileName);
        resp.setHeader("Content-Disposition","attachment;filename="+ URLEncoder.encode(fileName,"UTF-8"));
//        4. 获取下载文件的输入流
        FileInputStream fileInputStream =new FileInputStream(realPath);
//        5. 创建缓冲区
        int len = 0;
        byte[] buffer = new byte[1024];
//        6. 获取OutputStream对象
        ServletOutputStream out = resp.getOutputStream();
//        7. 将FileOutputStream流写入到bufer缓冲区
        while((len=fileInputStream.read(buffer))!=-1){
            out.write(buffer,0,len);
        }
        fileInputStream.close();
        out.close();
//        8. 使用OutputStream将缓冲区中的数据输出到客户端！
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}

```

```xml
<servlet>
      <servlet-name>FileServlet</servlet-name>
      <servlet-class>com.kuang.servlet.FileServlet</servlet-class>
    </servlet>
  <servlet-mapping>
    <servlet-name>FileServlet</servlet-name>
    <url-pattern>/fs</url-pattern>
  </servlet-mapping>
```

### 3. 实现重定向

![image-20210125190322378](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210125190322.png)

常见场景:

- 用户登录

```java
 void sendRedirect(String var1) throws IOException;
```

测试：

```java
package com.kuang.servlet;

public class RedirectServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.sendRedirect("/img");//重定向
    /*
    resp. setHeader("Location","/r/img");
    resp. setstatus (302);
    */
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}

```



```xml
    <servlet>
        <servlet-name>RedirectServlet</servlet-name>
        <servlet-class>com.kuang.servlet.RedirectServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>RedirectServlet</servlet-name>
        <url-pattern>/reditect</url-pattern>
    </servlet-mapping>
```

**面试题：请你聊聊重定向和转发的区别？**

- **forward（转发）**：307

是服务器请求资源,服务器直接访问目标地址的URL,把那个URL的响应内容读取过来,然后把这些内容再发给浏览器.浏览器根本不知道服务器发送的内容从哪里来的,因为这个跳转过程实在服务器实现的，并不是在客户端实现的所以客户端并不知道这个跳转动作，所以它的地址栏还是原来的地址.

- **redirect（重定向）**：302

是服务端根据逻辑,发送一个状态码,告诉浏览器重新去请求那个地址.所以地址栏显示的是新的URL.

**转发是服务器行为，重定向是客户端行为。**

相同点：页面都会跳转

不同点：

- 请求转发时：url不会产生变化
- 重定向：url会产生变化


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200505163953854.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbGxfbG92ZQ==,size_16,color_FFFFFF,t_70)
**index.jsp**

```jsp
<html>
<body>
<h2>Hel1o World!</h2>
《%--这里提交的路径,需要寻找到项目的路径--%>
<%--${pageContext. request, contextPath}代表当前的项目--%>
<form action="${pageContext. request.contextPath}/login" method="get">
    用户名: <input type="text" name="username"> <br>
    密码: <input type="password" name="password"> <br>
    <input type="submit">
</form>

</body>
</html>
```

**RequestTest.java**

```java
package com.kuang.servlet;

public class RequestTest extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //处理方求
        String username = req.getParameter( "username");
        String password =req.getParameter( "password");

        System.out.println(username+":"+password);

        resp.sendRedirect("/success.jsp");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}

```

**重定向页面success.jsp**

```html
<%@ page contentType="text/html; charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h1>success</h1>
</body>
</html>

```

**web.xml配置**

```xml
 <servlet>
        <servlet-name>requset</servlet-name>
        <servlet-class>com.kuang.servlet.RequestTest</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>requset</servlet-name>
      	  <url-pattern>/login</url-pattern>
    </servlet-mapping>
```

**导入依赖的jar包**

```xml
    <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>4.0.1</version>
    </dependency>
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>javax.servlet.jsp-api</artifactId>
      <version>2.3.3</version>
    </dependency>

  </dependencies>
```

## 6.7、HttpServletRequest

`HttpServletRequest`代表客户端的请求,用户通过`Http`协议访问服务器, `HTTP`请求中的所有信息会被封装到`HttpServletRequest`,通过这个`HttpServletRequest`的方法,获得客户端的所有信息;

获取前端传递的参数、请求转发

```java
        String username = req.getParameter("username"); // 从前端传来的变量
        String password = req.getParameter("password");
        String[] hobbys = req.getParameterValues("hobbys"); // 从前端传来的数组
```

前端页面传来数据

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>登录</title>
</head>
<body>
<div style="text-align: center">
    <form action="${pageContext.request.contextPath}/login" method="post">
        用户名：<input type="text" name="username"><br>
        密码：<input type="password" name="password"><br>
        爱好：
        <input type="checkbox" name="hobbys" value="女孩">女孩
        <input type="checkbox" name="hobbys" value="代码">代码
        <input type="checkbox" name="hobbys" value="唱歌">唱歌
        <input type="checkbox" name="hobbys" value="电影">电影
        <br>
        <input type="submit">
    </form>

</div>
</body>
</html>

```

获取前端传过来的数据

```java
package com.kuang.servlet;

public class LoginServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
      	req.setCharacterEncoding("utf-8");
      	resp.setCharacterEncoding("utf-8");
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        String[] hobbys = req.getParameterValues("hobbys");
        System.out.println("==========");
        //后台接收中文乱码问题
        System. out.println(username);
        System. out.println(password);
        System. out.println(Arrays.toString(hobbys));
        System. out.println("============");
        System. out.println(req.getContextPath());
        
        //通过请求转发
        //这里的/代表当前的web应用
        req.getRequestDispatcher("/success.jsp").forward(req,resp);
    }
}

```

在`web.xml`中配置映射，让服务器能通过映射找到`Java`文件

```xml
<servlet>
      <servlet-name>LoginServlet</servlet-name>
      <servlet-class>com.kuang.servlet.LoginServlet</servlet-class>
    </servlet>
  <servlet-mapping>
    <servlet-name>LoginServlet</servlet-name>
    <url-pattern>/login</url-pattern>
  </servlet-mapping>
```





# 7. Web服务

缺点：

- 加入服务器的动态web资源出现了错误，我们需要重新编写我们的**后台程序**,重新发布； 停机维护。

优点：

- Web页面可以**动态更新**，所有用户看到都不是同一个页面
- 它可以与**数据库交互** （数据持久化：注册，商品信息，用户信息…）

![image-20210122201724043](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210122201724.png)

## 7.1 现有技术

**ASP**:

- 微软：国内最早流行的就是ASP；
- 在HTML中嵌入了VB的脚本， ASP + COM；
- 在ASP开发中，基本一个页面都有几千行的业务代码，页面极其换乱
- 维护成本高！
- C#
- IIS

**php：**

- PHP开发速度很快，功能很强大，跨平台，代码很简单 （70% , WP）
- 无法承载大访问量的情况（局限性）

**JSP/Servlet :**

B/S：浏览器和服务器

C/S: 客户端和服务器

- sun公司主推的B/S架构
- 基于Java语言的 (所有的大公司，或者一些开源的组件，都是用Java写的)
- 可以承载三高（高并发、高可用、高性能）问题带来的影响；
- 语法像ASP ， ASP–--转----->JSP , 加强市场强度；

## 7.2 状态响应码

> 200：请求响应成功 200
>
> 3xx：请求重定向
>
> - 重定向：你重新到我给你新位置去；
>
> 4xx：找不到资源 404
>
> - 资源不存在；
>
> 5xx：服务器代码错误 500 502:网关错误

## 7.3 Cookie

客户端-------------------------------服务端

1. 服务端给客户端一个 信件，客户端下次访问服务端带上信件就可以了； **`cookie`**
2. 服务器登记你来过了，下次你来的时候我来匹配你； **`seesion`**

获取`cookies`列表：` Cookie[] cookies = req.getCookies();`

设置`cookie`：`resp.addCookie(cookie);`

**`Cookie`获取上次访问浏览器的时间**

```java
package com.kuang.servlet;

public class CookieDemo01 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //服务器给客户端发一个cookie 告你你来的时间，封装成一个信件，你下次来带来，我就知道你来了

        //解决中文乱码
        req.setCharacterEncoding("utf-8");
        resp.setCharacterEncoding("utf-8");
        resp.setContentType("text/html;charset=utf-8");
        
        // 获取输出对象
        Writer out = resp.getWriter();
        out.write("上次访问时间: ");

        // 获取访问时间cookie
        Cookie[] cookies = req.getCookies();
        for (int i = 0; cookies != null && i < cookies.length; i++) {

            Cookie cookie = cookies[i];
            if (cookie.getName().equals("lastAccess")) {
                String value = cookie.getValue();
                Date date = new Date(Long.parseLong(value));

                out.write(date.toString());
            }
        }

		// 设置最新的访问时间cookie
        Cookie cookie = new Cookie("lastAccess", System.currentTimeMillis() + "");
        // 设置cookie有效时间 单位：秒
        cookie.setMaxAge(3600);
        // 设置cookie有效路径
        cookie.setPath(req.getContextPath());
        //System.out.println(request.getContextPath());
        //System.out.println(this.getServletContext().getContextPath());
        resp.addCookie(cookie);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}

```

```xml
<servlet>
      <servlet-name>CookieDemo01</servlet-name>
      <servlet-class>com.kuang.servlet.CookieDemo01</servlet-class>
    </servlet>
  <servlet-mapping>
    <servlet-name>CookieDemo01</servlet-name>
    <url-pattern>/c1</url-pattern>
  </servlet-mapping>
```

![image-20210125211112759](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210125211112.png)



![image-20210126111418829](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126111418.png)

1. 从请求中拿到cookie信息
2. 服务器响应给客户端cookie

```java
Cookie[] cookies = req.getCookies(); //获得Cookie
cookie.getName(); //获得cookie中的key
cookie.getValue(); //获得cookie中的vlaue
new Cookie("lastLoginTime", System.currentTimeMillis()+""); //新建一个cookie
cookie.setMaxAge(24*60*60); //设置cookie的有效期
resp.addCookie(cookie); //服务器响应给客户端一个cookie
```

**cookie：一般会保存在本地的 用户目录下 appdata；**

![image-20210125211236088](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210125211236.png)

一个网站cookie是否存在上限！**聊聊细节问题**

- 一个Cookie只能保存**一个信息**；
- 一个web站点可以给浏览器发送**多个cookie**，**最多存放20个cookie**；
- Cookie**大小有限制4kb**；
- **300个cookie浏览器上限**

**删除Cookie；**

- 不设置有效期，关闭浏览器，自动失效；
- 设置有效期时间为 0 ；

## 7.4 Session（重点）

![image-20210126111619517](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126111619.png)
什么是`Session`：

- 服务器会给每一个用户（浏览器）创建一个`Seesion`对象；
- 一个`Seesion`独占一个浏览器，只要浏览器没有关闭，这个`Session`就存在；
- 用户登录之后，整个网站它都可以访问！–------> 保存用户的信息；保存购物车的信息……

`Session`和`cookie`的区别：

- `Cookie`是把用户的数据写给用户的浏览器，浏览器保存 （可以保存多个）
- `Session`把用户的数据写到用户独占`Session`中，服务器端保存 （保存重要的信息，减少服务器资源的浪费）
- `Session`对象由服务创建；

使用场景：

- 保存一个登录用户的信息；
- 购物车信息；
- 在整个网站中经常会使用的数据，我们将它保存在`Session`中；

使用`Session`：

得到`Session`: `HttpSession session = req.getSession();`
给`Session`中存东西,可以存很多东西，如变量，对象` session.setAttribute("name",new Person("秦疆",1));`
获取`Session`的ID： `String sessionId = session.getId();`

**使用Session存入对象，并获取当前默认创建的session ID**

```java
package com.kuang.servlet;

public class SessionDemo01 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        
        //解决乱码问题
        req.setCharacterEncoding("UTF-8");
        resp.setCharacterEncoding("UTF-8");
        resp.setContentType("text/html;charset=utf-8");
        
        //得到Session
        HttpSession session = req.getSession();
        //给Session中存东西,可以存很多东西，如变量，对象
        session.setAttribute("name",new Person("秦疆",1));
        //获取Session的ID
        String sessionId = session.getId();

        //判断Session是不是新创建
        if (session.isNew()){
            resp.getWriter().write("session创建成功,ID:"+sessionId);
        }else {
            resp.getWriter().write("session以及在服务器中存在了,ID:"+sessionId);
        }

        //Session创建的时候做了什么事情；
//        Cookie cookie = new Cookie("JSESSIONID",sessionId);
//        resp.addCookie(cookie);

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}

```

![image-20210126105518076](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126105518.png)

**获取Session存入的对象**

```java
package com.kuang.servlet;

public class SessionDemo02 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //解决乱码问题
        req.setCharacterEncoding("UTF-8");
        resp.setCharacterEncoding("UTF-8");
        resp.setContentType("text/html;charset=utf-8");

        //得到Session
        HttpSession session = req.getSession();

        Person person = (Person) session.getAttribute("name");

        System.out.println(person.toString());

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}

```



![image-20210126110336166](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126110336.png)

最后要在`web.xml`中添加配置

```xml
<servlet>
        <servlet-name>SessionDemo01</servlet-name>
        <servlet-class>com.kuang.servlet.SessionDemo01</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>SessionDemo01</servlet-name>
        <url-pattern>/s1</url-pattern>
    </servlet-mapping>

    <servlet>
        <servlet-name>SessionDemo02</servlet-name>
        <servlet-class>com.kuang.servlet.SessionDemo02</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>SessionDemo02</servlet-name>
        <url-pattern>/s2</url-pattern>
    </servlet-mapping>
```



**删除会话：注销**

```java
package com.kuang.servlet;

   public class SessionDemo03 extends HttpServlet {
   @Override
   protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       //解决乱码问题
       req.setCharacterEncoding("UTF-8");
       resp.setCharacterEncoding("UTF-8");
       resp.setContentType("text/html;charset=utf-8");

       //得到Session
       HttpSession session = req.getSession();
       session.removeAttribute("name");
   		//删除session
       session.invalidate();
   }

   @Override
   protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       super.doPost(req, resp);
   }
   }
```

**自动定时注销**

```xml
<!--设置Session默认的失效时间-->
<session-config>
    <!--15分钟后Session自动失效，以分钟为单位-->
    <session-timeout>15</session-timeout>
</session-config>

```



# 8、JSP

Java Server Pages ： Java服务器端页面，也和Servlet一样，用于动态Web技术！

最大的特点：

- 写JSP就像在写HTML
- 区别：
  - HTML只给用户提供静态的数据
  - JSP页面中可以嵌入JAVA代码，为用户提供动态数据；

## 8.1 JSP原理

思路：JSP到底怎么执行的！

- 代码层面没有任何问题

- 服务器内部工作

  tomcat中有一个work目录；

  IDEA中使用Tomcat的会在IDEA的tomcat中生产一个work目录

  ![在这里插入图片描述](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126111950.png)

  ![image-20210126112133357](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126112133.png)

  我电脑的地址：

  **C:\Users\Administrator\AppData\Local\JetBrains\IntelliJIdea2020.3\tomcat\4c2427d8-18f1-41a5-9c10-3480b51c0536\work\Catalina\localhost\javaweb_01_maven01_war\org\apache\jsp**

  发现页面转变成了Java程序！
  ![在这里插入图片描述](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126112152.png)

**浏览器向服务器发送请求，不管访问什么资源，其实都是在访问Servlet！**

JSP最终也会被转换成为一个Java类！

**JSP 本质上就是一个Servlet**   帮助简化了Servlet的使用

```java
//初始化
  public void _jspInit() {
      
  }
//销毁
  public void _jspDestroy() {
  }
//JSPService
  public void _jspService(.HttpServletRequest request,HttpServletResponse response)
```

1. 判断请求

2. 内置一些对象

   ```java
   final javax.servlet.jsp.PageContext.pageContext;  //页面上下文
   javax.servlet.http.HttpSession session = null;    //session
   final javax.servlet.ServletContext application;   //applicationContext
   final javax.servlet.ServletConfig config;         //config
   javax.servlet.jsp.JspWriter out = null;           //out
   final java.lang.Object page = this;               //page：当前
   HttpServletRequest request                        //请求
   HttpServletResponse response                      //响应
   ```

3. 输出页面前增加的代码

   ```java
   response.setContentType("text/html");       //设置响应的页面类型
   pageContext = _jspxFactory.getPageContext(this, request, response,
          null, true, 8192, true);
   _jspx_page_context = pageContext;
   application = pageContext.getServletContext();
   config = pageContext.getServletConfig();
   session = pageContext.getSession();
   out = pageContext.getOut();
   _jspx_out = out;
   ```

4. 以上的这些个对象我们可以在JSP页面中直接使用！

![image-20210126112949285](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126112949.png)

在JSP页面中；

只要是 JAVA代码就会原封不动的输出；

![image-20210126113206794](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126113206.png)

![image-20210126113222123](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126113222.png)

如果是HTML代码，就会被转换为：

```java
out.write("<html>\r\n");
```

这样的格式，输出到前端！

## 8.2 JSP依赖

在`maven`的`pom.xml`中添加

```xml
<!--        Servlet依赖-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
        </dependency>
<!--        JSP依赖-->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>2.3.3</version>
        </dependency>
<!--JSTL表达式依赖-->
        <dependency>
            <groupId>javax.servlet.jsp.jstl</groupId>
            <artifactId>jstl-api</artifactId>
            <version>1.2</version>
        </dependency>
<!--standard标签库-->
        <dependency>
            <groupId>taglibs</groupId>
            <artifactId>standard</artifactId>
            <version>1.1.2</version>
        </dependency>
```

## 8.3 JSP语法

**JSP声明**：`<%! declaration; [ declaration; ]+ ... %>`

示例：

```jsp
  <%!
    int a = 10;
    static {System.out.print(10);}
    public void ahang(){int b = 20;}
  %>
```



**JSP表达式**：只能写一个表达式`<%= 表达式 %>`

`  <%= new java.util.Date()%>`

**JSP注释**`<%-- 该部分注释在网页中不会被显示--%> `

**JSP脚本**

```jsp
  <%
    int sum = 0;
    for (int i = 1; i <=100 ; i++) {
      sum+=i;
    }
    out.println("<h1>Sum="+sum+"</h1>");
  %>
```

```jsp
  <%--在代码嵌入HTML元素--%>
  <%
    for (int i = 0; i < 5; i++) {
  %>
    <h1>Hello,World  <%=i%> </h1>
  <%
    }
  %>
```

**EL表达式**： `${ }` [使用教程](https://blog.csdn.net/w_linux/article/details/79850223)

- 获取数据
- 执行运算
- 获取web开发的常用对象

**JSP指令**

| **指令**           | **描述**                                                  |
| :----------------- | :-------------------------------------------------------- |
| <%@ page ... %>    | 定义页面的依赖属性，比如脚本语言、error页面、缓存需求等等 |
| <%@ include ... %> | 包含其他文件                                              |
| <%@ taglib ... %>  | 引入标签库的定义，可以是自定义标签                        |

`<%@ page errorPage="errorpage/500.jsp"%>`

**JSP行为**

`<jsp:include page="error.jsp" flush="true"/>`



**经过转化为Java文件后：**

所在位置：`C:\Users\105\.IntelliJIdea2018.2\system\tomcat\Unnamed_JavaWeb\work\Catalina\localhost\ROOT\org\apache\jsp\index_jsp.java`

```java
public final class index_jsp extends org.apache.jasper.runtime.HttpJspBase{
    // <%! 声明 %>声明的变量方法进入Java类的变量
    int a = 10;
    static {System.out.print(10);}
    public void ahang(){
        int b = 20;
    }
    ...
    // 这里面是页面显示的内容    
 	try {
      response.setContentType("text/html;charset=UTF-8");
      pageContext = _jspxFactory.getPageContext(this, request, response,
      			null, true, 8192, true);
      _jspx_page_context = pageContext;
      application = pageContext.getServletContext();
      config = pageContext.getServletConfig();
      session = pageContext.getSession();
      out = pageContext.getOut();
      _jspx_out = out;

      out.write("\n");
      out.write("\n");
   // HTML页面代码直接通过out.write("html代码")     
      out.write("<html>\n");
      out.write("  <head>\n");
      out.write("    <title>$Title$</title>\n");
      out.write("  </head>\n");
    //  <%= 表达式%>  
      out.print( new java.util.Date());

   //  <% 脚本 %>   
    int sum = 0;
    for (int i = 1; i <=100 ; i++) {
      sum+=i;
    }
    out.println("<h1>Sum="+sum+"</h1>");
 
	// <% 嵌套的脚本 %>
    for (int i = 0; i < 5; i++) {
      out.write("\n");
      out.write("  <h1>Hello,World  ");
      out.print(i);
      out.write(" </h1>\n");
      out.write("  ");
    }
        
     // JSP指令<%@ page errorPage="errorpage/500.jsp"%>
   org.apache.jasper.runtime.JspRuntimeLibrary.include(request, response, "error.jsp", out, true);
  
        
    // JSP行为动作<jsp:include page="error.jsp" flush="true"/>
      out.write("<html>\r\n");
      out.write("<head>\r\n");
      out.write("    <title>Title</title>\r\n");
      out.write("</head>\r\n");
      out.write("<body>\r\n");
      out.write("<h1>账号或密码不对，正在跳转登录界面。。。</h1><br>\r\n");
      out.write("<meta http-equiv=\"Refresh\" content=\"3;/index.jsp\">\r\n");
      out.write("<a href=\"/index.jsp\">或点此手动跳转登录页面</a>\r\n");
      out.write("</body>\r\n");
      out.write("</html>\r\n");
    }  
}     
```

## 8.4 定制错误页面

先建好错误页面，然后在`web.xml`中添加映射

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                             http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0"
         metadata-complete="true">

        <error-page>
            <error-code>404</error-code>
            <location>/errorpage/404.jsp</location>
        </error-page>
</web-app>
```

## 8.5 九大内置对象

| **对象**    | **描述**                                                     |
| :---------- | :----------------------------------------------------------- |
| request     | **HttpServletRequest**类的实例，【存东西】                   |
| response    | **HttpServletResponse**类的实例                              |
| out         | **PrintWriter**类的实例，用于把结果输出至网页上              |
| session     | **HttpSession**类的实例，【存东西】                          |
| application | **ServletContext**类的实例，与应用上下文有关，【存东西】     |
| config      | **ServletConfig**类的实例                                    |
| pageContext | **PageContext**类的实例，提供对JSP页面所有对象以及命名空间的访问，【存东西】 |
| page        | 类似于Java类中的this关键字                                   |
| Exception   | **Exception**类的对象，代表发生错误的JSP页面中对应的异常对象 |

### 8.5.1 数据存取

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <%
        pageContext.setAttribute("name1","秦疆1号"); //保存的数据只在一个页面中有效
        request.setAttribute("name2","秦疆2号"); //保存的数据只在一次请求中有效，请求转发会携带这个数据
        session.setAttribute("name3","秦疆3号"); //保存的数据只在一次会话中有效，从打开浏览器到关闭浏览器
        application.setAttribute("name4","秦疆4号");  //保存的数据只在服务器中有效，从打开服务器到关闭服务器
    %>

<%
    String name1 = (String) pageContext.findAttribute("name1");
    String name2 = (String) pageContext.findAttribute("name2");
    String name3 = (String) pageContext.findAttribute("name3");
    String name4 = (String) pageContext.findAttribute("name4");
    String name5 = (String) pageContext.findAttribute("name5");
%>

<%--使用输出 --%>
<h1>取出的值为：</h1>
<h1>${name1}</h1>
<h1>${name2}</h1>
<h1>${name3}</h1>
<h1>${name4}</h1>
<h1><%= name5%></h1>

</body>
</html>

```

![image-20210126123136297](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126123136.png)

### 8.5.2 请求转发

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

    <%
        pageContext.forward("/index.jsp");
        request.getRequestDispatcher("/index.jsp").forward(request,response);
    %>
</body>
</html>

```





![image-20210126123732111](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126123732.png)

![在这里插入图片描述](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126155513.png)

`request`：客户端向服务器发送请求，产生的数据，用户看完就没用了，比如：新闻，用户看完没用的！

`session`：客户端向服务器发送请求，产生的数据，用户用完一会还有用，比如：购物车；

`application`：客户端向服务器发送请求，产生的数据，一个用户用完了，其他用户还可能使用，比如：聊天数据；

# 9. MVC三层架构

- 什么是MVC： Model view Controller 模型、视图、控制器

## 9.1 以前的架构

![image-20210126184638367](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126184638.png)

用户直接访问控制层，控制层就可以直接操作数据库；

```java
servlet--CRUD-->数据库
弊端：程序十分臃肿，不利于维护  
servlet的代码中：处理请求、响应、视图跳转、处理JDBC、处理业务代码、处理逻辑代码

架构：没有什么是加一层解决不了的！
程序猿调用

          数据库
          JDBC （实现该接口）

Mysql    Oracle     SqlServer    ....（不同厂商）
```

## 9.2 MVC三层架构

![image-20210126184959830](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126185000.png)

**Model**

- 业务处理 ：业务逻辑（Service）
- 数据持久层：CRUD （Dao - 数据持久化对象）

**View**

- 展示数据
- 提供链接发起Servlet请求 （a，form，img…）

**Controller （Servlet）**

- 接收用户的请求 ：（req：请求参数、Session信息….）

- 交给业务层处理对应的代码

- 控制视图的跳转

  ```java
  登录--->接收用户的登录请求--->处理用户的请求（获取用户登录的参数，username，password）---->交给业务层处理登录业务（判断用户名密码是否正确：事务）--->Dao层查询用户名和密码是否正确-->数据库
  ```

# 10 Filter 过滤器处理乱码

比如 Shiro安全框架技术就是用Filter来实现的

Filter：过滤器 ，用来过滤网站的数据；

- 处理中文乱码
- 登录验证….

（比如用来过滤网上骂人的话）

![](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126185525.png)
Filter开发步骤：

1. 导包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.kuang</groupId>
    <artifactId>javaweb-05-filter</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>2.3.3</version>
        </dependency>
</project>
```

**中文乱码解决**

1. 编写过滤器
   1. 导包不要错 **（注意）**

![[(img-HHsC3JBD-1588757845420)(JavaWeb.assets/1568425162525.png)]](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126193130.png)

显示页面有中文，默认会乱码

```java
package com.kuang.servlet;

public class ShowServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("你好呀，世界！");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                             http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0"
         metadata-complete="true">

    <servlet>
      <servlet-name>ShowServlet</servlet-name>
      <servlet-class>com.kuang.servlet.ShowServlet</servlet-class>
    </servlet>
  <servlet-mapping>
    <servlet-name>ShowServlet</servlet-name>
    <url-pattern>/Servlet/Show</url-pattern>
  </servlet-mapping>
</web-app>
```

![image-20210126193841207](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126193841.png)

2.实现filer：

实现Filter接口，重写对应的方法即可

==注意：一定要重写`init()`和`destroy()`方法，即使为空也需要显示表示==，否则报错

```java
  public class CharacterEncodingFilter implements Filter {
  
      //初始化：web服务器启动，就以及初始化了，随时等待过滤对象出现！
      public void init(FilterConfig filterConfig) throws ServletException {
          System.out.println("CharacterEncodingFilter初始化");
      }
  
      //Chain : 链
      /*
      1. 过滤中的所有代码，在过滤特定请求的时候都会执行
      2. 必须要让过滤器继续同行
          chain.doFilter(request,response);
       */
      public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
          request.setCharacterEncoding("utf-8");
          response.setCharacterEncoding("utf-8");
          response.setContentType("text/html;charset=UTF-8");
  
          System.out.println("CharacterEncodingFilter执行前....");
          chain.doFilter(request,response); //让我们的请求继续走，如果不写，程序到这里就被拦截停止！
          System.out.println("CharacterEncodingFilter执行后....");
      }
  
      //销毁：web服务器关闭的时候，过滤器会销毁
      public void destroy() {
          System.out.println("CharacterEncodingFilter销毁");
      }
  }
  
```

3.在web.xml中配置 Filter

```xml
 <servlet>
      <servlet-name>ShowServlet</servlet-name>
      <servlet-class>com.kuang.servlet.ShowServlet</servlet-class>
    </servlet>
  <servlet-mapping>
    <servlet-name>ShowServlet</servlet-name>
    <url-pattern>/Servlet/Show</url-pattern>
  </servlet-mapping>
<!--    访问/Show就不会通过服务器-->
    <servlet-mapping>
        <servlet-name>ShowServlet</servlet-name>
        <url-pattern>/Show</url-pattern>
    </servlet-mapping>

    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>com.kuang.filter.CharacterEncodingFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <!--只要是 /servlet的任何请求，会经过这个过滤器-->
        <url-pattern>/Servlet/*</url-pattern>
        <!--<url-pattern>/*</url-pattern>-->
        <!-- 别偷懒写个 /* -->
    </filter-mapping>
```

![image-20210126195104343](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126195104.png)

# 11. Listener监听统计人数

实现一个监听器的接口；（有n种监听器）

1. 编写一个监听器

   实现监听器的接口…

   **统计网站在线人数**

   ```java
   package com.kuang.listener;
   
   public class OnlineCountListener implements HttpSessionListener {
       //创建session监听： 看你的一举一动
       //一旦创建Session就会触发一次这个事件！
       @Override
       public void sessionCreated(HttpSessionEvent httpSessionEvent) {
           ServletContext servletContext = httpSessionEvent.getSession().getServletContext();
   
           System.out.println(httpSessionEvent.getSession().getId());
   
           Integer onlineCount = (Integer) servletContext.getAttribute("OlineCount");
   
           if(onlineCount==null){
               onlineCount = new Integer(1);
           }else{
               int count = onlineCount.intValue();
               onlineCount = new Integer(count++);
           }
           servletContext.setAttribute("OlineCount",onlineCount);
       }
   
       //销毁session监听
       //一旦销毁Session就会触发一次这个事件！
       @Override
       public void sessionDestroyed(HttpSessionEvent httpSessionEvent) {
           ServletContext servletContext= httpSessionEvent.getSession().getServletContext();
   
           Integer onlineCount = (Integer) servletContext.getAttribute("OnlineCount");
   
           if (onlineCount==null){
               onlineCount = new Integer(0);
           }else {
               int count = onlineCount.intValue();
               onlineCount = new Integer(count-1);
           }
   
           servletContext.setAttribute("OnlineCount",onlineCount);
       }
   }
   
   
   /*
       Session销毁：
       1. 手动销毁  getSession().invalidate();
       2. 自动销毁
        */
   ```

2. web.xml中注册监听器

   ```xml
   <!--注册监听器-->
   <listener>
       <listener-class>com.kuang.listener.OnlineCountListener</listener-class>
   </listener>
   ```

3. 看情况是否使用！

4. ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>Title</title>
   </head>
   <body>
   <h1>当前在线人数：<span><%=this.getServletConfig().getServletContext().getAttribute("OlineCount")%></span></h1>
   </body>
   </html>
   
   ```

5. ![image-20210126201536381](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210126201536.png)

# 12. 登录权限过滤

用户登录之后才能进入主页！用户注销后就不能进入主页了！

登录流程：

1. 用户通过`index.jsp`也就是默认站点登录

2. 输入用户密码正确后进入`http://localhost:8080/sys/success.jsp`登录成功页面，点击页面上**注销**后即退出登录返回至`http://localhost:8080/index.jsp`默认界面

3. 输入用户密码错误后进入`http://localhost:8080/error.jsp`登录错误页面，三秒后自动跳转至主页面

4. 如果直接输入url为：`http://localhost:8080/sys/success.jsp`会因为没有权限而跳转至`http://localhost:8080/error.jsp`错误页面

   ![image-20210127004231313](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210127004231.png)

   ![image-20210127004302056](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210127004302.png)

   ![image-20210127004309504](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210127004309.png)


**实现思路：**

1. 在首页实现post提交用户名密码，交给后台验证

2. 后端通过实现一个`Loginservlet.java`来对数据验证，如果数据匹配则给该请求创建一个`Session`来给过滤器使用，然后跳转成功页面；如果不匹配则直接跳转错误页面

3. 注销实现一个`LogOutservlet.java`来删除该`Session`，然后重定向到错误页面

4. 为了防止用户在没有登录时，直接访问`http://localhost:8080/sys/success.jsp`页面，需要添加一个过滤器在filter中添加`SysFilter.java`来判断请求是否携带`Session`，如果没有携带则跳转至错误页面，否则通过放行

   

![image-20210127004208149](https://typora-wenjiuzhou.oss-cn-beijing.aliyuncs.com/20210127004208.png)

1. 首页`index.jsp`

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<form action="/servlet/login" method="post">
    <input type="text" name="username">
    <input type="submit">
</form>
</body>
</html>
```

2. 登录成功后的页面`success.jsp`

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>Title</title>
   </head>
   <body>
   <h1>主页</h1>
   <p> <a href="/servlet/logout">注销</a> </p>
   </body>
   </html>
   
   ```

3. 登录出错或没有权限的错误页面

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>Title</title>
   </head>
   <body>
   <h1>账号或密码不对没有权限，3秒后跳转登录界面。。。</h1><br>
   <meta http-equiv="Refresh" content="3;/index.jsp">
   <a href="/index.jsp">或点此手动跳转登录页面</a>
   </body>
   </html>
   ```

4. 登录后通过后端`Loginservlet`判断

```java
package com.kuang.servlet;

public class Loginservlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //获取前端传来的参数
        String username = req.getParameter("username");
        if(username.equals("admin")){
            // 给当前Session设一个特定属性，方便后面filter判断
            req.getSession().setAttribute("USER_SESSION",req.getSession().getId());
            // 登录成功后跳转至成功页面并给设定一个Session
            resp.sendRedirect("/sys/success.jsp");
        }else{
            // 登录失败跳转错误页面
            resp.sendRedirect("/error.jsp");
        }
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
    }
}

```

5. 登出注销通过后端`LogoutServlet.java`来实现

```java
package com.kuang.servlet;

public class LogoutServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Object user_session = req.getSession().getAttribute("USER_SESSION");
        // 用户注销，删除session，跳转首页
        if(user_session!=null){
            req.getSession().removeAttribute("USER_SESSION");
            resp.sendRedirect("/Login.jsp");
        }else{
            resp.sendRedirect("/Login.jsp");
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}

```

6. 通过过滤实现权限管理

```java
package com.kuang.filter;

public class SysFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        HttpServletResponse httpServletResponse = (HttpServletResponse) servletResponse;

        if((httpServletRequest.getSession().getAttribute("USER_SESSION"))==null){
            httpServletResponse.sendRedirect("/error.jsp");
        }
        filterChain.doFilter(servletRequest,servletResponse);
    }

    @Override
    public void destroy() {

    }
}

```

7. 最后在`web.xml`中添加映射的配置

```xml
<servlet>
        <servlet-name>LoginServlet</servlet-name>
        <servlet-class>com.kuang.servlet.Loginservlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>LoginServlet</servlet-name>
        <url-pattern>/servlet/login</url-pattern>
    </servlet-mapping>

    <servlet>
        <servlet-name>LogoutServlet</servlet-name>
        <servlet-class>com.kuang.servlet.LogoutServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>LogoutServlet</servlet-name>
        <url-pattern>/servlet/logout</url-pattern>
    </servlet-mapping>

	<filter>
        <filter-name>SysFilter</filter-name>
        <filter-class>com.kuang.filter.SysFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>SysFilter</filter-name>
        <!--只要是 /sys的任何请求，会经过这个过滤器-->
        <url-pattern>/sys/*</url-pattern>
        <!--<url-pattern>/*</url-pattern>-->
        <!-- 别偷懒写个 /* -->
    </filter-mapping>

```






































