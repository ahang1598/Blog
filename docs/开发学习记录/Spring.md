[TOC]

# 1. 基本概念

## 1.1 优点
- Spring是一个开源的免费的框架（容器）
- Spring是一个轻量级的，非侵入式的框架
- 控制反转（IOC `Inversion of Control` 和面向切面编程（AOP `Aspect Oriented Programming`）
- 支持事务的处理，对框架整合的支持！

## 1.2 组成
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201223175912704.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70)
- Spring-Core：Core包是框架的最基础部分，并提供依赖注入（Dependency Injection）管理Bean容器功能。这里的基础概念是BeanFactory，它提供对Factory模式的经典实现来消除对程序性单例模式的需要，并真正地允许你从程序逻辑中分离出依赖关系和配置。
- Spring-Context：核心模块的BeanFactory使Spring成为一个容器，而上下文模块使它成为一个框架。这个模块扩展了BeanFactory的概念，增加了消息、事件传播以及验证的支持。另外，这个模块提供了许多企业服务，例如电子邮件、JNDI访问、EJB集成、远程以及时序调度（scheduling）服务。也包括了对模版框架例如Velocity和FreeMarker集成的支持。
- Spring-Aop：Spring在它的AOP模块中提供了对面向切面编程的丰富支持。例如方法拦截器（servletListener ,controller....）和切点，可以有效的防止代码上功能的耦合，这个模块是在Spring应用中实现切面编程的基础。Spring的AOP模块也将元数据编程引入了Spring。使用Spring的元数据支持，你可以为你的源代码增加注释，指示Spring在何处以及如何应用切面函数。

- Spring-Dao：使用JDBC经常导致大量的重复代码，取得连接、创建语句、处理结果集，然后关闭连接、旧代码中迁移自定义工具类JDBCUtil 也让开发变得繁琐。Spring的Dao模块对传统的JDBC进行了抽象，还提供了一种比编程性更好的声明性事务管理方法。
- Spring-Web：Web上下文模块建立于应用上下文模块之上，提供了WEB开发的基础集成特性，例如文件上传。另外，这个模块还提供了一些面向服务支持。利用Servlet listeners进行IOC容器初始化和针对Web的applicationcontext。

- Spring Web MVC：(Model-View-Controller)Spring为构建Web应用提供了一个功能全面的MVC框架。它提供了一种清晰的分离模型，在领域模型代码和web form之间。并且，还可以借助Spring框架的其他特性。
- Spring-ORM：关系映射模块，ORM包为流行的“关系/对象”映射APIs提供了集成层，包括JDO，Hibernate和iBatis（MyBatis）。通过ORM包，可以混合使用所有Spring提供的特性进行“对象/关系”映射，方便开发时小组内整合代码。

![77160637A5F74F17A02B64416B25F466](3CCBBFCB8B57457FA398B19A0A60765E)

## 1.3 基本案例

创建`UseDao`接口

`UseDaoImpl`,`UseDaoMysqlImpl`,`UseDaoOrcalImpl`实现接口

`UseService`业务接口

`UseServiceImpl`业务实现接口

`applicationContext.xml`Spring的配置文件

```java
public interface UseDao {
    public void use();
}

public class UseDaoImpl implements UseDao {
    public void use(){
        System.out.println("use dao...");
    }
}

public class UseDaoMysqlImpl implements UseDao{
    public void use(){
        System.out.println("use mysql...");
    }
}

public class UseDaoOrcalImpl implements UseDao{
    public void use(){
        System.out.println("use Orcal...");
    }
}

public interface UseService {
    public void use();
}

public class UseServiceImpl implements UseService {
    private UseDao useDao;

    // 通过set注入，IOC实现反转
    public void setUseDao(UseDao useDao){
        this.useDao = useDao;
    }

    public void use(){
        useDao.use();
    }
}

// 测试结果，现在是由我们自行控制创建对象 , 把主动权交给了调用者
    @Test
    public void test1() {
        UseService useService = new UseServiceImpl();
        ((UseServiceImpl) useService).setUseDao(new UseDaoMysqlImpl());
        useService.use();

        ((UseServiceImpl) useService).setUseDao(new UseDaoOrcalImpl());
        useService.use();
    }
```



现在通过配置Spring的`applicationContext.xml`文件来控制反转

**控制反转是一种通过描述（XML或注解）并通过第三方去生产或获取特定对象的方式。在Spring中实现控制反转的是IoC容器，其实现方法是依赖注入（Dependency Injection,DI）**

```xml
在maven的pom.xml配置中添加依赖
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-webmvc</artifactId>
   <version>5.1.10.RELEASE</version>
</dependency>
```

```java
// 1. 在pojo中编写一个普通的类，无参构造类型
public class Hello {
   private String name;

   public String getName() {
       return name;
  }
   public void setName(String name) {
       this.name = name;
  }

   public void show(){
       System.out.println("Hello,"+ name );
  }
}
```

2. 在`applicationContext.xml`中配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    // id任意起的唯一标识名字，表示该类对象； class为调用类的全路径名；property属性中name为类中的变量名，value给该变量赋值
    <bean id="hello" class="com.ahang.demo.pojoo.Hello">
        <property name="str" value="nihao"></property>
    </bean>
</beans>    
```

3. 最后在测试类中测试

   ```java
       @Test
       public void test1() {
           // 找到配Spring的配置文件
           ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
           // 通过getBean(id)获取配置文件中的对应的类对象
           Hello h = (Hello)context.getBean("hello");
           System.out.println(h.toString());
           System.out.println(h.getStr());
       }
   ```

   - Hello 对象是谁创建的 ? hello 对象是由Spring创建的
   - Hello 对象的属性是怎么设置的 ? hello 对象的属性是由Spring容器设置的

   这个过程就叫控制反转 :

   - **控制 : 谁来控制对象的创建 , 传统应用程序的对象是由程序本身控制创建的 , 使用Spring后 , 对象是由Spring来创建的**
   - **反转 : 程序本身不创建对象 , 而变成被动的接收对象 .**

   依赖注入 : 就是利用set方法来进行注入的.

   IOC是一种编程思想，由主动的编程变成被动的接收

4. 通过在配置中添加来控制使用`ServiceImpl`

```xml
    <bean id="useDaoMysql" class="com.ahang.demo.dao.UseDaoMysqlImpl"></bean>
    <bean id="useDaoOrcal" class="com.ahang.demo.dao.UseDaoOrcalImpl"></bean>
    <bean id="useServiceImpl" class="com.ahang.demo.service.UseServiceImpl">
        <property name="useDao" ref="useDaoMysql"></property>
    </bean>
```

```java
    @Test
    public void test2(){
        ApplicationContext con = new ClassPathXmlApplicationContext("applicationContext.xml");
        UseServiceImpl useServiceImpl = (UseServiceImpl) con.getBean("useServiceImpl");
        useServiceImpl.use();
    }
```

## 1.4 基本配置

### 1.4.1 scope范围

1）当`scope`的取值为`singleton`时

      `Bean`的实例化个数：1个
    
      `Bean`的实例化时机：当`Spring`核心文件被加载时，实例化配置的`Bean`实例
    
      `Bean`的生命周期：

对象创建：当应用加载，创建容器时，对象就被创建了

对象运行：只要容器在，对象一直活着

对象销毁：当应用卸载，销毁容器时，对象就被销毁了

2）当`scope`的取值为`prototype`时

      `Bean`的实例化个数：多个
    
      `Bean`的实例化时机：当调用`getBean()`方法时实例化`Bean`

对象创建：当使用对象时，创建新的对象实例

对象运行：只要对象在使用中，就一直活着

对象销毁：当对象长时间不用时，被 `Java` 的垃圾回收器回收了

```xml
    <bean id="hello" class="com.ahang.demo.pojo.Hello" scope="singleton"> </bean>
```

```java
    @Test
    public void test5(){
        ApplicationContext con = new ClassPathXmlApplicationContext("applicationContext.xml");
        Hello hello =(Hello) con.getBean("hello");
        Hello hello1 = (Hello) con.getBean("hello");
        System.out.println(hello == hello1);  // true scope范围为单例时，声明了多个其实都是同一个对象同一个地址
    }
```



```xml
    <bean id="hello" class="com.ahang.demo.pojo.Hello" scope="prototype"> </bean>
```

```java
    @Test
    public void test5(){
        ApplicationContext con = new ClassPathXmlApplicationContext("applicationContext.xml");
        Hello hello =(Hello) con.getBean("hello");
        Hello hello1 = (Hello) con.getBean("hello");
        System.out.println(hello == hello1);  // false, scope范围为多例时，声明了多个则有多个地址不同的对象
    }
```

### 1.4.2 生命周期

`init-method`：指定类中的初始化方法名称

`destroy-method`：指定类中销毁方法名称

`hello`类中声明`init()`和`destory()`方法：

```xml
    <bean id="hello" class="com.ahang.demo.pojo.Hello" scope="singleton" init-method="init" destroy-method="destory"> </bean>
```

### 1.4.3 有参构造

1. 通过指定参数名称配置

```xml
    <bean id="hello" class="com.ahang.demo.pojo.Hello" scope="singleton" >
        <constructor-arg name="str" value="haha" />
    </bean>
```

2. 通过指定参数位置

```xml
    <bean id="hello" class="com.ahang.demo.pojoo.Hello">
        <constructor-arg index="0" value="haha" />
    </bean>
```

### 1.4.4 别名名称

```xml
    <bean id="hello" class="com.ahang.demo.pojo.Hello" name="h1 h2,h3;h4"></bean>  <!--设置的name可以通过空格,;等隔开-->

	<alias name="hello" alias="hello2"/>
```

### 1.4.5 import导入其他配置

对于团队开发多个合作，有多个`xml`的配置文件，最后需要汇总到一个中`applicationContext.xml`

```xml
<import resource="beans1.xml"/>
<import resource="beans2.xml"/>
<import resource="beans3.xml"/>
```

## 1.6 依赖注入
Dependency Injection

依赖 : 指Bean对象的创建依赖于容器 . Bean对象的依赖资源 .

注入 : 指Bean对象所依赖的资源 , 由容器来设置和装配 .
### 1.6.1 构造器注入
通过构造函数录入参数

### 1.6.2 Set注入

在pojo中测试类1
```java
public class Address {
    private String address;
    public String getAddress(){ return address; }
    public void setAddress(String address){ this.address = address; }
}
```
对于多种类型的数据看如何注入
```java
public class InjectDemo {
    private String name;
    private Address address;
    private String[] str;
    private List<String> list;
    private Map<Integer,String> map;
    private Set<String> set;
    private String kong;
    private Properties info;
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    ...getXXX...setXXX
```
在`applicationContext.xml`中对于不同类型数据注入方式：
```xml
    <bean id="addr" class="com.ahang.demo.pojo.Address">
        <property name="address" value="China"></property>
    </bean>

    <bean id="inject" class="com.ahang.demo.pojo.InjectDemo">
        <property name="name" value="ahang"></property>  // 常量
        <property name="address" ref="addr"></property>  // 引用其他类
        // 数组
        <property name="str">
            <array>
                <value>str1</value>
                <value>str2</value>
                <value>str3</value>
            </array>
        </property>
        
        // 列表
        <property name="list">
            <list>
                <value>list1</value>
                <value>list2</value>
                <value>list3</value>
            </list>
        </property>
        
        // map表
        <property name="map">
            <map>
                <entry key="1" value="123"/>
                <entry key="2" value="234"/>
                <entry key="3" value="345"/>
            </map>
        </property>

        // set表
        <property name="set">
            <set>
                <value>set1</value>
                <value>set2</value>
                <value>set3</value>
            </set>
        </property>
        
        // 空null类型数据
        <property name="kong"><null></null></property>

        // properties类型
        <property name="info">
            <props>
                <prop key="id">1234</prop>
                <prop key="name">haha</prop>
                <prop key="gender">man</prop>
            </props>
        </property>
    </bean>
```

### 1.6.3 p命名和c命名注入
需要在配置文件中添加：
```xml
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
```
```java
public class User {
    private String name;
    private int age;

    public User(){}

    public User(String name, int age){
        this.name = name;
        this.age = age;
    }
}
```
具体使用：
- p命名直接对变量注入
- c命名需要注意添加有参构造方法才能使用
```xml
    <bean id="user" class="com.ahang.demo.pojo.User" p:name="ahang" p:age="10"></bean>

    <bean id="user1" class="com.ahang.demo.pojo.User" c:name="haha" c:age="20"></bean>
```

## 1.7 自动装配
环境满足`Spring`的`bean`依赖条件，会在上下文中寻找对应的`bean`

Spring中bean有三种装配机制，分别是：
- 在xml中显式配置；
- 在java中显式配置；
- 隐式的bean发现机制和自动装配

**推荐不使用自动装配xml配置, 而使用注解**

```java
public class Cat {
    public void bark(){
        System.out.println("miao~");
    }
}

public class Dog {
    public void bark(){
        System.out.println("wang~");
    }
}

public class User {
    private String name;
    private int age;
    private Cat cat;
    private Dog dog;
    public Cat getCat() {return cat;}
    public void setCat(Cat cat) {this.cat = cat;}
    ...getXXX...setXXX...
    
```
### 1.7.1 `byType`自动装配
只要上下文中存在`User`类中定义的`bean`类型：如`User`中存在`Cat`和`Dog`两个引用类，而上文中存在定义好的`bean`类型为`Cat`和`Dog`且唯一，`bean`的`id`可以随意命名。

如果存在两个`Dog`类型的`bean`则会报错，如

`<bean id="dog1" class="com.ahang.demo.pojo.Dog"></bean>`  
`<bean id="dog2" class="com.ahang.demo.pojo.Dog"></bean>`
```xml
    <bean id="cat1" class="com.ahang.demo.pojo.Cat" ></bean>
    <bean id="dog" class="com.ahang.demo.pojo.Dog"></bean>
    <bean id="user2" class="com.ahang.demo.pojo.User" autowire="byType">
        <constructor-arg name="age" value="10"></constructor-arg>
        <constructor-arg name="name" value="kaka"></constructor-arg>
        <!--<property name="cat" ref="cat"></property>-->
        <!--<property name="dog" ref="dog"></property>-->
        <property name="age" value="30"></property>
    </bean>
```

### 1.7.2 `byName`自动装配
上下文需要存在`Bean`的`id`等同于`User`中的方法`setXXX`中`XXX`才能自动装配

则可以存在多个相同类型的`bean`，但是`id`需要唯一对应相同，如：

`<bean id="dog" class="com.ahang.demo.pojo.Dog"></bean>`

`<bean id="dog2" class="com.ahang.demo.pojo.Dog"></bean>`

```xml
    <bean id="cat" class="com.ahang.demo.pojo.Cat" ></bean>
    <bean id="cat1" class="com.ahang.demo.pojo.Cat" ></bean> 
    // 可以存在相同Cat，但是id不同，只要存在一个id和User类中setXXX相同即可
    <bean id="dog" class="com.ahang.demo.pojo.Dog"></bean>
    <bean id="dog2" class="com.ahang.demo.pojo.Dog"></bean>
    <bean id="user2" class="com.ahang.demo.pojo.User" autowire="byName">
        <constructor-arg name="age" value="10"></constructor-arg>
        <constructor-arg name="name" value="kaka"></constructor-arg>
        <!--<property name="cat" ref="cat"></property>-->
        <!--<property name="dog" ref="dog"></property>-->
        <property name="age" value="30"></property>
    </bean>
```

## 1.8 使用注解装配注入
在配置文件中加入注解支持
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd ">

    <context:annotation-config></context:annotation-config>
</beans>
```
在配置中不用手动的添加装配
```xml
    <bean id="cat1" class="com.ahang.demo.pojo.Cat"></bean>
    <bean id="dog1" class="com.ahang.demo.pojo.Dog"></bean>
    <bean id="dog2" class="com.ahang.demo.pojo.Dog"></bean>
    
    <bean id="use" class="com.ahang.demo.pojo.Use">
        <property name="str" value="haha"></property>
    </bean>
```
在Java文件中,通过在对象上面添加注解或者在set方法上添加注解`@Autowired`，就可以根据类型自动装配，也可以通过添加`@Qualifier("id")`指定id名称装配
```java
public class Use {
    @Autowired
    private Cat cat;
    
    @Autowired
    @Qualifier("dog1")
    private Dog dog;
    
    private String str;
    ...
}    
```




## 1.9 基于Java类配置

## 1.10 代理模式

### 10.1静态代理

**静态代理角色分析**

- 抽象角色 : 一般使用接口或者抽象类来实现---租房`Rent`
- 真实角色 : 被代理的角色---房主`Host`
- 代理角色 : 代理真实角色 ; 代理真实角色后 , 一般会做一些附属的操作----代理中介`Proxy`
- 客户 : 使用代理角色来进行一些操作 ----`Client`
- 房主`Host`想出租房`Rent`,房主进行了出租
- 但是房主又不想天天带人看房`seeHouse`，就把钥匙`Proxy(Host host)`给了中介`Proxy`
- 然后中介`Proxy`就自己又发布了出租告示`Rent`
- 这时候有客户`Client`想要租房，那么就得带着出租告示`Rent`去找中介`Proxy`来租房子
- 中介在出租时就会同时看房`seeHouse`和出租`Rent`

`Rent`租房告示

```java
//抽象角色：租房
public interface Rent {
   public void rent();
}
```

`Host`房主

```java
//真实角色: 房东，房东要出租房子
public class Host implements Rent{
   public void rent() {
       System.out.println("房屋出租");
  }
}
```

中介`Proxy`

```java
//代理角色：中介
public class Proxy implements Rent {

   private Host host;
   public Proxy() { }
   public Proxy(Host host) {
       this.host = host;
  }

   //租房
   public void rent(){
       seeHouse();
       host.rent();
  }
   //看房
   public void seeHouse(){
       System.out.println("带房客看房");
  }
}

```

客户`Client.java`

```java
//客户类，一般客户都会去找代理！
public class Client {
   public static void main(String[] args) {
       //房东要租房
       Host host = new Host();
       //中介帮助房东
       Proxy proxy = new Proxy(host);

       //你去找中介！
       proxy.rent();
  }
}
```

**静态代理的好处:**

- 可以使得我们的真实角色更加纯粹 . 不再去关注一些公共的事情 .
- 公共的业务由代理来完成 . 实现了业务的分工 ,
- 公共业务发生扩展时变得更加集中和方便 .

缺点 :

- 类多了 , 多了代理类 , 工作量变大了 . 开发效率降低 .

我们想要静态代理的好处，又不想要静态代理的缺点，所以 , 就有了动态代理 !

### 10.2动态代理

- 动态代理的角色和静态代理的一样 .
- 动态代理的代理类是动态生成的 . 静态代理的代理类是我们提前写好的
- 动态代理分为两类 : 一类是基于接口动态代理 , 一类是基于类的动态代理
- - 基于接口的动态代理----JDK动态代理
  - 基于类的动态代理–cglib
  - 现在用的比较多的是 javasist 来生成动态代理 . 百度一下javasist
  - 我们这里使用JDK的原生代码来实现，其余的道理都是一样的！、

**JDK的动态代理需要了解两个类**

核心 : InvocationHandler 和 Proxy 

先定义一个接口`Rent`和一个实现该接口得`Host`类

```java
public interface Rent {
    public void rent();
}

public class Host implements Rent {
    @Override
    public void rent() {
        System.out.println("rental hosting");
    }
}
```

然后定义一个动态代理类`ProxyInvocationHandler`实现代理`Rent`

> `Object invoke(Object proxy, Method method, Object[] args)`；
> //参数
> //`proxy` - 调用该方法的代理实例
> //`method` -所述方法对应于调用代理实例上的接口方法的实例。方法对象的声明类将是该方法声明的接口，它可以是代理类继承该方法的代理接口的超级接口。
> //`args` -包含的方法调用传递代理实例的参数值的对象的阵列，或null如果接口方法没有参数。原始类型的参数包含在适当的原始包装器类的实例中，例如`java.lang.Integer`或`java.lang.Boolean `。

```java
package com.ahang.pojo;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class ProxyInvocationHandler implements InvocationHandler {
    private Rent rent;

    public void setRent(Rent rent){
        this.rent = rent;
    }

    public Object getProxy(){
        return Proxy.newProxyInstance(this.getClass().getClassLoader(), rent.getClass().getInterfaces(), this);
    }
    // 处理代理实例，并返回结果
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 动态代理的本质：反射机制实现        
        seeHouse();
        Object invoke = method.invoke(rent, args);
        return invoke;
    }

    public void seeHouse(){
        System.out.println("see house");
    }
}
```

最后测试`Client`类

```java
        Host host = new Host();
        ProxyInvocationHandler pih = new ProxyInvocationHandler();
        pih.setRent(host);
        Rent proxy =(Rent) pih.getProxy();
        proxy.rent();
```

### 10.3 通用动态类

对于`ProxyInvocationHandler`

```java
public class ProxyInvocationHandler implements InvocationHandler {

    private Object target;

    public void setRent(Object target) {
        this.target = target;
    }

    public Object getProxy() {
        return Proxy.newProxyInstance(this.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
    }

    // 处理代理实例，并返回结果
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 动态代理的本质：发射机制实现
        log("调用了方法 " + method.getName());
        Object result = method.invoke(target, args);
        return result;
    }

    public void log(String msg) {
        System.out.println(msg);
    }
}
```

在使用的时候对不同的接口和实现类替换即可

```java
        Host host = new Host();
        ProxyInvocationHandler pih = new ProxyInvocationHandler();
        pih.setTarget(host);
        Rent proxy = (Rent) pih.getProxy();
        proxy.rent();
```

```java
        UserServiceImpl userService = new UserServiceImpl();
        ProxyInvocationHandler pih = new ProxyInvocationHandler();
        pih.setRent(userService);
        UserService proxy = (UserService) pih.getProxy();
        proxy.add();
        proxy.del();
```









## 1.11 AOP

### 1.11.1 什么是AOP

AOP（Aspect Oriented Programming）意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率

### 11.2.Aop在Spring中的作用

提供声明式事务；允许用户自定义切面

以下名词需要了解下：

- 横切关注点：跨越应用程序多个模块的方法或功能。即是，与我们业务逻辑无关的，但是我们需要关注的部分，就是横切关注点。如日志 , 安全 , 缓存 , 事务等等 …
- 切面（ASPECT）：横切关注点 被模块化 的特殊对象。即，它是一个类。
- 通知（Advice）：切面必须要完成的工作。即，它是类中的一个方法。
- 目标（Target）：被通知对象。
- 代理（Proxy）：向目标对象应用通知之后创建的对象。
- 切入点（PointCut）：切面通知 执行的 “地点”的定义。
- 连接点（JointPoint）：与切入点匹配的执行点

![](https://img-blog.csdnimg.cn/20200613182141599.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDIwNzQwMw==,size_16,color_FFFFFF,t_70)

SpringAOP中，通过Advice定义横切逻辑，Spring中支持5种类型的Advice:

![](https://img-blog.csdnimg.cn/20200613182147344.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDIwNzQwMw==,size_16,color_FFFFFF,t_70)

即 Aop 在 不改变原有代码的情况下 , 去增加新的功能 

### 1.11.3 具体实现

添加依赖

```xml
<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
<dependency>
   <groupId>org.aspectj</groupId>
   <artifactId>aspectjweaver</artifactId>
   <version>1.9.4</version>
</dependency>
```



`execution`表达式的使用`execution(访问修饰符（可省略） 返回值类型 包.包.类.方法名(参数) )`如

`<aop:before method="printLog" pointcut="execution(public void com.huge.service.impl.AccountServiceImpl.save())"></aop:before>`

| 符号 | 类型匹配模式                                                 |
| ---- | ------------------------------------------------------------ |
| *    | 匹配任何数量字符；比如模式 (*,String) 匹配了一个接受两个参数的方法，第一个可以是任意类型，第二个则必须是String类型 |
| ..   | 匹配任何数量字符的重复，如在类型模式中匹配任何数量子包；而在方法参数模式中匹配任何数量参数，可以使零到多个。 |
| +    | 匹配指定类型的子类型；仅能作为后缀放在类型模式后边。         |

| 符号       | 参数匹配模式：                                               |
| ---------- | ------------------------------------------------------------ |
| ()         | 匹配了一个不接受任何参数的方法，                             |
| (..)       | 匹配了一个接受任意数量参数的方法（零或者更多）。             |
| (*)        | 匹配了一个接受一个任何类型的参数的方法。                     |
| (*,String) | 匹配了一个接受两个参数的方法，第一个可以是任意类型， 第二个则必须是String类型。 |

示例：
```java
// 省略访问修饰符
void com.huge.service.impl.AccountServiceImpl.save()

// 返回值使用通配符，表示可以返回任意值
* com.huge.service.impl.AccountServiceImpl.save()

// 包名可以使用通配符，表示任意包。但是有几级包，就需要写几个*.
* *.*.*.*.AccountServiceImpl.save()

// 包名可以使用..表示当前包及其子包
* *..AccountServiceImpl.save()

//定义在service 包里,及其子类的任意类的任意方法
* com.huge.service.*.*()

// 类名和方法名都可以使用*来实现通配
* *..*.*()

// 所有包下的service,及其子包
* *.service..*()

// IAccountService 若为接口，则为接口中的任意方法及其所有实现类中的任意,方法；若为类，则为该类及其子类中的任意方法
* com.huge.service.IAccountService+.*()

// 以s开头的任意方法
* com.huge.service.*.s*()
```

```java
// 基本类型直接写名称 int
* com.huge.service.*.*(int)

// 方法不带参数的
* com.huge.service.*.*()

// 可以使用..表示有无参数均可，有参数可以是任意类型
* com.huge.service.*.*(..)

// 两个参数,第一个是String,第二个任意,三个参数不行
execution(* joke(String,*)))

// 两个参数,第一个String,第二个个数和类型不限
execution(* joke(String,..)))

// 全通配写法：
* *..*.*(..)
```

#### 1.11.3.1 **方法一**

首先编写我们的业务接口和实现类

```java
public interface UserService {

   public void add();

   public void delete();

   public void update();

   public void search();

}
public class UserServiceImpl implements UserService{

   @Override
   public void add() {
       System.out.println("增加用户");
  }

   @Override
   public void delete() {
       System.out.println("删除用户");
  }

   @Override
   public void update() {
       System.out.println("更新用户");
  }

   @Override
   public void search() {
       System.out.println("查询用户");
  }
}

```

然后去写我们的增强类 , 我们编写两个 , 一个前置增强 一个后置增强

```java
public class Log implements MethodBeforeAdvice {

   //method : 要执行的目标对象的方法
   //objects : 被调用的方法的参数
   //Object : 目标对象
   @Override
   public void before(Method method, Object[] objects, Object o) throws Throwable {
       System.out.println( o.getClass().getName() + "的" + method.getName() + "方法被执行了");
  }
}
public class AfterLog implements AfterReturningAdvice {
   //returnValue 返回值
   //method被调用的方法
   //args 被调用的方法的对象的参数
   //target 被调用的目标对象
   @Override
   public void afterReturning(Object returnValue, Method method, Object[] args,Object target) throws Throwable {
       System.out.println("执行了" + target.getClass().getName()
       +"的"+method.getName()+"方法,"
       +"返回值："+returnValue);
  }
}
```

最后去spring的文件中注册 , 并实现aop切入实现 , 注意导入约束 .

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:aop="http://www.springframework.org/schema/aop"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">

   <!--注册bean-->
   <bean id="userService" class="com.kuang.service.UserServiceImpl"/>
   <bean id="log" class="com.kuang.log.Log"/>
   <bean id="afterLog" class="com.kuang.log.AfterLog"/>

   <!--aop的配置-->
   <aop:config>
       <!--切入点 expression:表达式匹配要执行的方法-->
       <aop:pointcut id="pointcut" expression="execution(* com.kuang.service.UserServiceImpl.*(..))"/>
       <!--执行环绕; advice-ref执行方法 . pointcut-ref切入点-->
       <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
       <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
   </aop:config>

</beans>
```

测试

```jav
public class MyTest {
   @Test
   public void test(){
       ApplicationContext context = newClassPathXmlApplicationContext("beans.xml");
       UserService userService = (UserService) context.getBean("userService");
       userService.search();
  }
}
```


#### 1.11.3.2 **第二种方式**:**自定义类来实现Aop**

目标业务类不变依旧是userServiceImpl

第一步 : 写我们自己的一个切入类

```java
public class DiyPointcut {

   public void before(){
       System.out.println("---------方法执行前---------");
  }
   public void after(){
       System.out.println("---------方法执行后---------");
  }
   
}
12345678910
```

去spring中配置

```xml
<!--第二种方式自定义实现-->
<!--注册bean-->
<bean id="diy" class="com.kuang.config.DiyPointcut"/>

<!--aop的配置-->
<aop:config>
   <!--第二种方式：使用AOP的标签实现-->
   <aop:aspect ref="diy">
       <aop:pointcut id="diyPonitcut" expression="execution(* com.kuang.service.UserServiceImpl.*(..))"/>
       <aop:before pointcut-ref="diyPonitcut" method="before"/>
       <aop:after pointcut-ref="diyPonitcut" method="after"/>
   </aop:aspect>
</aop:config>
12345678910111213
```

测试：

```java
public class MyTest {
   @Test
   public void test(){
       ApplicationContext context = newClassPathXmlApplicationContext("beans.xml");
       UserService userService = (UserService) context.getBean("userService");
       userService.add();
  }
}
```



#### 1.11.3.3**第三种方式**:**使用注解实现**

第一步：编写一个注解实现的增强类

```java
package com.kuang.config;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class AnnotationPointcut {
   @Before("execution(* com.kuang.service.UserServiceImpl.*(..))")
   public void before(){
       System.out.println("---------方法执行前---------");
  }

   @After("execution(* com.kuang.service.UserServiceImpl.*(..))")
   public void after(){
       System.out.println("---------方法执行后---------");
  }

   @Around("execution(* com.kuang.service.UserServiceImpl.*(..))")
   public void around(ProceedingJoinPoint jp) throws Throwable {
       System.out.println("环绕前");
       System.out.println("签名:"+jp.getSignature());
       //执行目标方法proceed
       Object proceed = jp.proceed();
       System.out.println("环绕后");
       System.out.println(proceed);
  }
}
```

第二步：在Spring配置文件中，注册bean，并增加支持注解的配置

```xml
<!--第三种方式:注解实现-->
<bean id="annotationPointcut" class="com.kuang.config.AnnotationPointcut"/>
<aop:aspectj-autoproxy/>
```

> `aop:aspectj-autoproxy`：说明
>
> 通过aop命名空间的`<aop:aspectj-autoproxy />`声明自动为`spring`容器中那些配置`@aspect`J切面的bean创建代理，织入切面。当然，spring 在内部依旧采用`AnnotationAwareAspectJAutoProxyCreator`进行自动代理的创建工作，但具体实现的细节已经被`<aop:aspectj-autoproxy />`隐藏起来了
>
> `<aop:aspectj-autoproxy />`有一个`proxy-target-class`属性，默认为false，表示使用jdk动态代理织入增强，当配为`<aop:aspectj-autoproxy  poxy-target-class="true"/>`时，表示使用CGLib动态代理技术织入增强。不过即使`proxy-target-class`设置为false，如果目标类没有声明接口，则spring将自动使用`CGLib`动态代理。

# 2. Spring-MyBatis

MyBatis-Spring 会帮助你将 MyBatis 代码无缝地整合到 Spring 中。

[官方文档]: http://www.mybatis.org/spring/zh/index.html

## 2.2 基本配置(方式一)

1. 添加一个maven模块，在`pom.xml`中添加依赖包

```xml
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
        <spring.version>5.0.5.RELEASE</spring.version>
    </properties>

	<dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.6</version>
    </dependency>
	<!--添加Lombok来简化操作-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.20</version>
        </dependency>
```

2. 在数据库中添加表`user`

```mysql
mysql> select * from user;
+----+-------+
| id | name  |
+----+-------+
| 10 | ahang |
| 20 | gaga  |
+----+-------+
```

3. 在`com.ahang.pojo`包下添加`User`类

```java
package com.ahang.pojo;
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private int id;
    private String name;
}
```

4. 在`com.ahang.Dao`包下添加三个文件`UserDao`接口，`UserDaoImpl`Java类，`UserDao.xml`配置文件

```java
public interface UserDao {
    List<User> getUser();
}
```

```java
public class UserDaoImpl implements UserDao {
    private SqlSessionTemplate sqlSessionTemplate;
	// 代理模式：代理创建数据库的连接
    public void setSqlSessionTemplate(SqlSessionTemplate sqlSessionTemplate){
        this.sqlSessionTemplate = sqlSessionTemplate;
    }

    public List<User> getUser(){
        UserDao mapper = sqlSessionTemplate.getMapper(UserDao.class);
        return mapper.getUser();
    }
}
```

`UserDao.xml`配置文件写入对数据库的具体操作

==注意事项：`namespace`中为接口的全类名==

==`resultType`:这里`users`时对`User`全类名的别名==

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.ahang.Dao.UserDao">  
    <select id="getUser" resultType="users">
        select * from user;
    </select>
</mapper>
```

5. 最后在`resource`中添加两个配置文件：`mybatis.xml`和`applicationContext.xml`

对于`mybatis.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--核心配置文件-->
<configuration>
    <!--给实体类起别名-->
    <typeAliases>
        <typeAlias type="com.ahang.pojo.User" alias="users"></typeAlias>
    </typeAliases>
</configuration>
```

对于`Spring`的`appplicationContext.xml`

对于数据库操作配置文件的绑定语句

可以使用具体的配置文件如下：`<property name="mapperLocations" value="classpath:com/ahang/Dao/UserDao.xml"></property> ` 

也可以修改为配置文件匹配:`classpath*:com/ahang/Dao/*.xml`

`classpath`：只会到你的class路径中查找找文件

`classpath*`不仅包含class路径，还包括jar文件中(class路径)进行查找，

否则会报错：`‘mapperLocations’ was specified but matching resources are not found.`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--配置数据源：数据源有非常多，可以使用第三方的，也可使使用Spring的-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
        <property name="username" value="root"/>
        <property name="password" value="6666"/>
    </bean>
    
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"></property>
        <property name="configLocation" value="classpath:mybatis.xml"></property> //mybatis核心配置文件
        <property name="mapperLocations" value="classpath:com/ahang/Dao/UserDao.xml"></property>  // 修改为对应的具体增删改查配置文件，也可以更改为`classpath*:com/ahang/Dao/*.xml`
    </bean>

    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory"></constructor-arg>
    </bean>

    <bean id="userDao" class="com.ahang.Dao.UserDaoImpl">  // 修改为对应的实现类
        <property name="sqlSessionTemplate" ref="sqlSession"></property>
    </bean>
</beans>
```

6. 测试

   ```java
       @Test
       public void test1(){
           ClassPathXmlApplicationContext con = new ClassPathXmlApplicationContext("applicationContext.xml");
           UserDao userDao = con.getBean("userDao", UserDao.class); 
           // 这里的第一个参数对应applicationContext.xml配置中的bean的id,而第二个参数对应接口的类
           List<User> user = userDao.getUser();
           for (User e : user) {
               System.out.println(e);
           }
       }
   ```

   

## 2.3 基本配置（方式二）

在原方式一的基础上修改两处，其余不变

1. 修改`UserDaoImpl`,通过继承`SqlSessionDaoSupport`来简化数据库连接注入

```java
public class UserDaoImpl extends SqlSessionDaoSupport implements UserDao {

    @Override
    public List<User> getUser() {
        return getSqlSession().getMapper(UserDao.class).getUser();
    }

    @Override
    public int insertUser(User user) {
        return getSqlSession().getMapper(UserDao.class).insertUser(user);
    }

    @Override
    public int deleteUser(int id) {
        return getSqlSession().getMapper(UserDao.class).deleteUser(id);
    }
}
```

2. 修改`applicationContext.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
           <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
           <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
           <property name="username" value="root"/>
           <property name="password" value="1598"/>
       </bean>
   
       <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
           <property name="dataSource" ref="dataSource"></property>
           <property name="configLocation" value="classpath:mybatis.xml"></property>
           <property name="mapperLocations" value="classpath:com/ahang/Dao/UserDao.xml"></property>
       </bean>
       
       <!--将原来的数据库连接过程简化了-->
           <!--<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">-->
           <!--<constructor-arg index="0" ref="sqlSessionFactory"></constructor-arg>-->
       <!--</bean>-->
   
       <!--<bean id="userDao" class="com.ahang.Dao.UserDaoImpl">-->
           <!--<property name="sqlSessionTemplate" ref="sqlSession"></property>-->
       <!--</bean>-->
   
       <bean id="userDao" class="com.ahang.Dao.UserDaoImpl">
           <property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
       </bean>
   </beans>
   ```

   

最后在测试

```java
    @Test
    public void test1(){
        ApplicationContext con = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserDao userDao = con.getBean("userDao", UserDao.class);
        List<User> user = userDao.getUser();
        for (User e : user) {
            System.out.println(e);
        }
    }

    @Test
    public void test2(){
        ApplicationContext con = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserDao userDao = con.getBean("userDao", UserDao.class);
        User user = new User(30, "haha");
        int i = userDao.insertUser(user);
        if(i > 0) System.out.println("insert successed!!!");
        else System.out.println("insert fail");
    }

    @Test
    public void test3(){
        ApplicationContext con = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserDao userDao = con.getBean("userDao", UserDao.class);
        int i = userDao.deleteUser(30);
        if(i > 0) System.out.println("delete successed!!!");
        else System.out.println("delete fail");
    }
```

## 2.3 配置事务

**事务四个属性ACID**

1. 原子性（atomicity）

   事务是原子性操作，由一系列动作组成，事务的原子性确保动作要么全部完成，要么完全不起作用

2. 一致性（consistency）

   一旦所有事务动作完成，事务就要被提交。数据和资源处于一种满足业务规则的一致性状态中

3. 隔离性（isolation）

   可能多个事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏

4. 持久性（durability）

   事务一旦完成，无论系统发生什么错误，结果都不会受到影响。通常情况下，事务的结果被写到持久化存储器中

这里的事务配置针对的是在实现接口的一个方法中所有操作具有事务性，也就是`UserDaoImpl`中的一个方法如：`public List<User> getUser() `,而不是指在测试方法中调用多个方法多种操作的事务。

先对`applicationContext.xml`中配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
        <property name="username" value="root"/>
        <property name="password" value="1598"/>
    </bean>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"></property>
        <property name="configLocation" value="classpath:mybatis.xml"></property>
        <property name="mapperLocations" value="classpath:com/ahang/Dao/UserDao.xml"></property>
    </bean>

    <bean id="userDao" class="com.ahang.Dao.UserDaoImpl">
        <property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
    </bean>
    
    
    <!-- 配置声明式事务 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!-- 结合aop实现事务的植入-->
    <!-- 配置事务通知 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!--这里的name就是指实现接口UserDaoImpl下的方法名称-->
            <tx:method name="getUser" propagation="REQUIRED"/>
            <tx:method name="delete" propagation="REQUIRED"/>
            <tx:method name="update" propagation="REQUIRED"/>
            <tx:method name="search" propagation="REQUIRED"/>
            <tx:method name="get" read-only="true"/>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>
    <!--配置事务切入，针对Dao包下的所有类的所有方法-->
    <aop:config>
        <aop:pointcut id="txPointCut" expression="execution(* com.ahang.Dao.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointCut"/>
    </aop:config>

</beans>
```



对接口实现类`UserDaoImpl`配置,针对第一个方法故意配置出错，在第一个方法中插入语句`mapper.insertUser(user)`正常，但是`mapper.deleteUser(10)`有问题

- 如果测试时没有配置事务，那么执行了`getUser()`方法后会先插入数据，然后删除数据时出错，但是最后输入还是插入成功了
- 如果配置了事务，那么会先插入数据，然后删除数据出错，但是由于事务的一致性，会回退至插入数据前，最后插入失败。

```java
public class UserDaoImpl extends SqlSessionDaoSupport implements UserDao {

    @Override
    public List<User> getUser() {
        User user = new User(15, "kaka");
        UserDao mapper = getSqlSession().getMapper(UserDao.class);
        int i = mapper.insertUser(user);
        if(i > 0) System.out.println("insert successed");
        mapper.deleteUser(10);
        return mapper.getUser();
    }

    @Override
    public int insertUser(User user) {
        return getSqlSession().getMapper(UserDao.class).insertUser(user);
    }

    @Override
    public int deleteUser(int id) {
        return getSqlSession().getMapper(UserDao.class).deleteUser(id);
    }
}
```

在配置`UserDao.xml`时第三条删除语句时，故意设置数据表`user`为`users`，那么在测试会出错

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.ahang.Dao.UserDao">
    <select id="getUser" resultType="users">
        select * from user;
    </select>

    <insert id="insertUser" parameterType="users">
        insert into user values( #{id}, #{name} );
    </insert>

    <delete id="deleteUser" parameterType="int">
        delete from users where id=#{id};
    </delete>
</mapper>
```

具体测试：

```java
    @Test
    public void test1(){
        ApplicationContext con = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserDao userDao = con.getBean("userDao", UserDao.class);
        List<User> user = userDao.getUser();
        for (User e : user) {
            System.out.println(e);
        }
    }
```



**spring事务传播特性：**

事务传播行为就是多个事务方法相互调用时，事务如何在这些方法间传播。spring支持7种事务传播行为：

针对的是`<tx:method name="*" propagation="REQUIRED"/>`中的`propagation`

- propagation_requierd：如果当前没有事务，就新建一个事务，如果已存在一个事务中，加入到这个事务中，这是最常见的选择。
- propagation_supports：支持当前事务，如果没有当前事务，就以非事务方法执行。
- propagation_mandatory：使用当前事务，如果没有当前事务，就抛出异常。
- propagation_required_new：新建事务，如果当前存在事务，把当前事务挂起。
- propagation_not_supported：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- propagation_never：以非事务方式执行操作，如果当前事务存在则抛出异常。
- propagation_nested：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与propagation_required类似的操作

Spring 默认的事务传播行为是 PROPAGATION_REQUIRED，它适合于绝大多数的情况。

假设 ServiveX#methodX() 都工作在事务环境下（即都被 Spring 事务增强了），假设程序中存在如下的调用链：Service1#method1()->Service2#method2()->Service3#method3()，那么这 3 个服务类的 3 个方法通过 Spring 的事务传播机制都工作在同一个事务中。

就好比，我们刚才的几个方法存在调用，所以会被放在一组事务当中！



























