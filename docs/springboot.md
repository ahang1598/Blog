# 1. spring与spring boot

## 1.1、为什么用SpringBoot



> Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".
>
> 能快速创建出生产级别的Spring应用



## 1.2、SpringBoot优点

- Create stand-alone Spring applications
  - 创建独立Spring应用

- Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files)
  - 内嵌web服务器

- Provide opinionated 'starter' dependencies to simplify your build configuration
  - 自动starter依赖，简化构建配置

- Automatically configure Spring and 3rd party libraries whenever possible
  - 自动配置Spring以及第三方功能

- Provide production-ready features such as metrics, health checks, and externalized configuration
  - 提供生产级别的监控、健康检查及外部化配置

- Absolutely no code generation and no requirement for XML configuration
  - 无代码生成、无需编写XML



SpringBoot是整合Spring技术栈的一站式框架

SpringBoot是简化Spring技术栈的快速开发脚手架



## 1.3、SpringBoot缺点

- 人称版本帝，迭代快，需要时刻关注变化
- 封装太深，内部原理复杂，不容易精通



- 微服务是一种架构风格
- 一个应用拆分为一组小型服务

- 每个服务运行在自己的进程内，也就是可独立部署和升级
- 服务之间使用轻量级HTTP交互

- 服务围绕业务功能拆分
- 可以由全自动部署机制独立部署

- 去中心化，服务自治。服务可以使用不同的语言、不同的存储技术

1）“微服务”[原文链接](http://martinfowler.com/articles/microservices.html)

2）“微服务”[译文链接](http://mp.weixin.qq.com/s?__biz=MjM5MjEwNTEzOQ==&mid=401500724&idx=1&sn=4e42fa2ffcd5732ae044fe6a387a1cc3#rd)



# 2. springboot入门

1.通过idea创建springboot

![springboot-2.1创建](img/springboot-2.1创建.png)



![springboot-2.1创建2](img/springboot-2.1创建2.png)

![springboot-2.1创建3](img/springboot-2.1创建3.png)













#  3.自动配置原理

##  3.1 依赖管理

默认的pom文件依赖父项目

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.2</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
```

父项目的pom中设定所有版本，本地就无须设置版本号

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

**修改本地版本号两种方式**

1. 通过在本地pom文件中设置`properties`（**推荐**）

```xml
    <properties>
        <mysql.version>5.1.43</mysql.version>
    </properties>
```

2. 在本地pom的依赖出设置

```xml
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.43</version>
        </dependency>
```



**开发导入starter场景启动器**

1、见到很多 `spring-boot-starter-* `： *就某种场景

 2、只要引入`starter`，这个场景的所有常规需要的依赖我们都自动引入 

3、`SpringBoot`所有支持的场景 [详细地址](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter )

4、见到的 ` *-spring-boot-starter`： 第三方为我们提供的简化开发的场景启动器。

 5、所有场景启动器最底层的依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

		<dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.2.0</version>
        </dependency>
```



## 3.2 自动配置

- 自动配好`SpringMVC`
  - 引入`SpringMVC`全套组件
  - 自动配好`SpringMVC`常用组件（功能）

- 自动配好`Web`常见功能，如：字符编码问题
  - `SpringBoot`帮我们配置好了所有web开发的常见场景

- 默认的包结构
  - 主程序所在包及其下面的所有子包里面的组件都会被默认扫描进来
  - 无需以前的包扫描配置
  - 想要改变扫描路径，`@SpringBootApplication(scanBasePackages="com.atguigu")`
    - 或者`@ComponentScan` 指定扫描路径

```java
@SpringBootApplication
等同于
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("com.atguigu.boot")
```

- 各种配置拥有默认值
  - 默认配置最终都是映射到某个类上，如：`MultipartProperties`
  - 配置文件的值最终会绑定每个类上，这个类会在容器中创建对象

- 按需加载所有自动配置项
  - 非常多的`starter`
  - 引入了哪些场景这个场景的自动配置才会开启
  - `SpringBoot`所有的自动配置功能都在` spring-boot-autoconfigure `包里面
- ....

## 3.3 容器组件

### **1、@Configuration注解**

- 作用：声明一个类为**配置类**（等同于我们以前的配置文件）
- 注意：
  - `配置类本身也是组件`（会作为一个对象放到IOC容器中）
  - springboot2中多了一个`proxyBeanMethods`属性
    - `proxyBeanMethods`属性的默认值是true
    - `proxyBeanMethods：代理bean的方法
    - 如果 `proxyBeanMethods = true`，**Full模式**，调用IOC中配置类的方法，多次调用的得到的对象`是同一个（单例的）`
      - 为true，为`代理对象调用方法`。SpringBoot总会检查这个组件是否在IOC容器中有。有的话直接拿，没有创建，保持组件单实例
    - 如果` proxyBeanMethods = false`,**Lite模式** 调用IOC中配置类的方法，多次调用的得到的对象`不是同一个（多例的）`
      - 为false，每个@Bean方法被调用多少次返回的组件都是**新创建的**
  - **Full模式与Lite模式** **可以解决组件间的依赖问题**
    - 示例:
      - 配置 类组件之间`无依赖`关系用`Lite多例模式`加速容器启动过程，减少判断，快
      - 配置类组件之间`有依赖`关系，`Full单例模式`，方法会被调用得到之前单实例组件，慢

```java
#############################Configuration使用示例############################
/**
 * 1、配置类里面使用@Bean标注在方法上给容器注册组件，默认也是单实例的
 * 2、配置类本身也是组件
 * 3、proxyBeanMethods：代理bean的方法
 *      Full(proxyBeanMethods = true)、【保证每个@Bean方法被调用多少次返回的组件都是单实例的】
 *      Lite(proxyBeanMethods = false)【每个@Bean方法被调用多少次返回的组件都是新创建的】
 *      组件依赖必须使用Full模式默认。其他默认是否Lite模式
 *
 */
@Configuration(proxyBeanMethods = false) //告诉SpringBoot这是一个配置类 == 配置文件
public class MyConfig {

    /**
     * Full:外部无论对配置类中的这个组件注册方法调用多少次获取的都是之前注册容器中的单实例对象
     * @return
     */
    @Bean //给容器中添加组件。以方法名作为组件的id。返回类型就是组件类型。返回的值，就是组件在容器中的实例
    public User user01(){
        User zhangsan = new User("zhangsan", 18);
        //user组件依赖了Pet组件
        zhangsan.setPet(tomcatPet());
        return zhangsan;
    }

    @Bean("tom")
    public Pet tomcatPet(){
        return new Pet("tomcat");
    }
}


################################@Configuration测试代码如下###################################
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("com.atguigu.boot")
public class MainApplication {

    public static void main(String[] args) {
        //1、返回我们IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);
        
        //2、查看容器里面的组件
        String[] names = run.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name);
        }
        
        //3、从容器中获取组件，如果两个相等，就说明为单实例的
        Pet tom01 = run.getBean("tom", Pet.class);
        Pet tom02 = run.getBean("tom", Pet.class);
        System.out.println("组件："+(tom01 == tom02));

        //4、其实配置类本身也相当于一个组件
        //com.atguigu.boot.config.MyConfig$$EnhancerBySpringCGLIB$$51f1e1ca@1654a892
        MyConfig bean = run.getBean(MyConfig.class);
        System.out.println(bean);

        //如果@Configuration(proxyBeanMethods = true)代理对象调用方法。SpringBoot总会检查这个组件是否在容器中有。有直接拿(单例)
        //如果 proxyBeanMethods = false,调用IOC中配置类的方法，多次调用的得到的对象 不是同一个（多例的）
        User user = bean.user01();
        User user1 = bean.user01();
        System.out.println(user == user1);
        
        //@Configuration(proxyBeanMethods = true) 用户的宠物是容器中的宠物
        //@Configuration(proxyBeanMethods = false) 用户的宠物是不是容器中的宠物
        User user01 = run.getBean("user01", User.class);
        Pet tom = run.getBean("tom", Pet.class);
        System.out.println("用户的宠物："+(user01.getPet() == tom));
0
    }
}
```

### 1、@Bean  注解

- 使用位置：  方法上
- 作用：给容器中添加组件。以**方法名作为组件的id**。返回类型就是组件类型。**返回的值**，就是组件在容器中的**实例**

- 属性：
  - 无属性的时候，默认方法名作为组件的id，可以使用id属性设置组件的id
  - 当只有这一个属性的时候，可以省略

- 注意：配置类里面使用@Bean标注在方法上给容器注册组件，默认也是单实例的



### 2、@Component、@Controller、@Service、@Repository  注解

- 使用位置：类上
- 作用：
  - 在学spring的注解方式创建bean对象组件，在这里同样适用



### 3、@ComponentScan  

- 使用位置：类上
- 作用：包扫描
  - 在学spring的注解方式，进行包扫描一样使用
  - 有value属性、basePackages属性： 都可以指定要扫描的包



### 4、@Import  

- 使用位置：类上（配置类、组件类）
- 作用：给容器中**导入多个组件**
  - 给容器中自动创建出这的组件
  - 默认组件的名字就是全类名



```java
//@Import({User.class, DBHelper.class})
// 给容器中自动创建出这两个类型的组件、默认组件的名字就是全类名
@Import({User.class, DBHelper.class})
@Configuration(proxyBeanMethods  = false) //告诉SpringBoot这是一个配置类 == 配置文件 
public class MyConfig {
}
```

[@Import ]() 高级用法： https://www.bilibili.com/video/BV1gW411W7wy?p=8



### 5、@Conditional  

- 作用：条件装配，满足Conditional指定的条件，则进行组件注入
- 位置：类上（配置类上）,方法上
  - 放在方法上，如果符合响应的要求，创建对应方法返回值的组件
  - 放到类（配置类）上，当符合要求的时候，创建配置类所有的组件，否则不创建配置类下的组件

- 属性：
  - name : 指定组件的名称（判断容器中是否有该名称的组件）
  - value : 指定组件的类型Class（判断容器中是否有该类型的组件）

- 注意：
  - @Conditional注解是一个根注解，他有许多的派生注解
- 派生注解：  
  - @ConditionalOnBean注解：当容器中存在指定的组件的时候，才会做某些事情
  - @ConditionalOnMissingBean注解：当容器中不存在指定的组件的时候，才会做某些事情
  - @ConditionalOnClass注解：当容器中有某一个类的时候，才会做某些事情
  - @ConditionalOnMissingClass注解：当容器中，没有某一个类的时候，才会做某些事情
  - @ConditionalOnResource注解：当容器中项目的类路径里面存在某些资源的时候，才会做某些事情
  - @ConditionalOnJava注解:是指定的java版本号的时候，才会做某些事情
  - @ConditionalOnWebApplication注解:当我们的应用是一个web应用的时候，才会做某些事情
  - @ConditionalOnNotWebApplication注解:当我们的应用不是一个web应用的时候，才会做某些事情
  - @ConditionalOnProperty注解：当配置文件配置了指定的属性的时候，才会做某些事情
  - @ConditionalOnSingleCandidate注解：指定的组件有一个实例或者有多个实例一个主实例的时候,才会做某些事情  
  - .....    
- ![img](https://cdn.nlark.com/yuque/0/2021/png/12563077/1615807324651-1e30c697-a926-49dd-9122-526eafe65497.png)



```java
=====================测试条件装配==========================
@Configuration(proxyBeanMethods = false) //告诉SpringBoot这是一个配置类 == 配置文件
//@ConditionalOnBean(name = "tom")
@ConditionalOnMissingBean(name = "tom")
public class MyConfig {


    /**
     * Full:外部无论对配置类中的这个组件注册方法调用多少次获取的都是之前注册容器中的单实例对象
     * @return
     */

    @Bean //给容器中添加组件。以方法名作为组件的id。返回类型就是组件类型。返回的值，就是组件在容器中的实例
    public User user01(){
        User zhangsan = new User("zhangsan", 18);
        //user组件依赖了Pet组件
        zhangsan.setPet(tomcatPet());
        return zhangsan;
    }

    @Bean("tom22")
    public Pet tomcatPet(){
        return new Pet("tomcat");
    }
}

public static void main(String[] args) {
        //1、返回我们IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);

        //2、查看容器里面的组件
        String[] names = run.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name);
        }

        boolean tom = run.containsBean("tom");
        System.out.println("容器中Tom组件："+tom);

        boolean user01 = run.containsBean("user01");
        System.out.println("容器中user01组件："+user01);

        boolean tom22 = run.containsBean("tom22");
        System.out.println("容器中tom22组件："+tom22);

    }
```

### 6. @ImportResource

原生配置文件引入

```xml
======================beans.xml=========================
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="haha" class="com.atguigu.boot.bean.User">
        <property name="name" value="zhangsan"></property>
        <property name="age" value="18"></property>
    </bean>

    <bean id="hehe" class="com.atguigu.boot.bean.Pet">
        <property name="name" value="tomcat"></property>
    </bean>
</beans>

```

```java
@ImportResource("classpath:beans.xml")
public class MyConfig {}

======================测试=================
        boolean haha = run.containsBean("haha");
        boolean hehe = run.containsBean("hehe");
        System.out.println("haha："+haha);//true
        System.out.println("hehe："+hehe);//true
```



## 3.4 配置绑定

- 如何使用Java读取到properties文件中的内容，并且把它封装到JavaBean中，以供随时使用；



```java
//使用java原生代码：（比较麻烦）
public class getProperties {
     public static void main(String[] args) throws FileNotFoundException, IOException {
         Properties pps = new Properties();
         pps.load(new FileInputStream("a.properties"));
         Enumeration enum1 = pps.propertyNames();//得到配置文件的名字
         while(enum1.hasMoreElements()) {
             String strKey = (String) enum1.nextElement();
             String strValue = pps.getProperty(strKey);
             System.out.println(strKey + "=" + strValue);
             //封装到JavaBean。
         }
     }
 }
```

### 1. @ConfigurationProperties

- 作用：绑定`properties`配置文件（主配置文件`application.properties`）中的内容，到`javabean`对象中
- 位置：类上（bean类上）
  - 注意：一般会把该类放到容器中，同时使用该注解`bean`配置参数
- 属性：
  - preifx属性：指定对应的`properties`中的前缀
  - 通过 前缀.属性，与类中的属性一一对应



### 2、`@EnableConfigurationProperties + @ConfigurationProperties `

**进行配置类的配置绑定**

- 配置类上：`@EnableConfigurationProperties ` 开启某个类属性配置功能，并且把该类自动注册到容器中
- 这时候如果该类上有`@ConfigurationProperties`注解，会进行自动的配置

- `@EnableConfigurationProperties`的属性写 一个类的`class`



```java
@SpringBootApplication
@EnableConfigurationProperties(Car.class)
//1、开启Car配置绑定功能
//2、把这个Car这个组件自动注册到容器中
public class MyConfig {
}

--------------------------------------------------------
@ConfigurationProperties(prefix = "mycar")
public class Car {
    private String brand;
    private Integer price;
    ....
}
-------------------------------------------------------
//application.properties配置文件
mycar.brand = BYD
mycar.price = 100000
```





### 3、`@Component+@ConfigurationProperties 进行bean对象类的配置绑定`

- bean类进行参数绑定
- **@Component注解把该类放到容器中**，@ConfigurationProperties 注解进行配置绑定



```java
//application.properties配置文件

mycar.brand = BYD
mycar.price = 100000


/**
 * 只有在容器中的组件，才会拥有SpringBoot提供的强大功能
 */
@Component
@ConfigurationProperties(prefix = "mycar")
public class Car {

    private String brand;
    private Integer price;

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public Integer getPrice() {
        return price;
    }

    public void setPrice(Integer price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return "Car{" +
                "brand='" + brand + '\'' +
                ", price=" + price +
                '}';
    }
}
```



### 4、@ConfigurationProperties  进行bean组件的配置绑定

- 把`@ConfigurationProperties注解 加到返回的Bean组件上`，**会把配置文件的信息，绑定到返回的bean组件上**

```java
//配置类
@Configuration
public class MyDataSourceConfig {
   @ConfigurationProperties("spring.datasource")//把配置文件的信息，绑定到返回的组件上
   @Bean
   public DataSource dataSource() {
      DruidDataSource druidDataSource = new DruidDataSource();
      //数据源的设置（这种方式写死了，最好使用配置文件的自动注入进来）
//    druidDataSource.setUrl("jdbc:mysql://localhost:3306/zhangyang?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8&useSSL=false");
//    druidDataSource.setUsername("root");
//    druidDataSource.setPassword("015718");
//    druidDataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
      return druidDataSource;
   }
}
```

### 5、使用Spring中的@Value注解，绑定Bean类的属性值

![img](https://cdn.nlark.com/yuque/0/2021/png/12563077/1615888823760-c5f1da07-ac4a-4ae8-9d3b-acb99f7e2446.png)



## 3.5 引导加载自动配置类



```java
//@SpringBootApplication相当于下面三个注解
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication{
    
}
```



### 1、@SpringBootConfiguration注解



代表当前是一个配置类



### 2、@ComponentScan注解



指定扫描哪些，Spring注解；



### 3、@EnableAutoConfiguration 注解



- 也是一个合成注解：由下面两个注解合成



```
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}
```



#### 1、@AutoConfigurationPackage



- 自动配置包？指定了默认的包规则
- 点开该注解的源码如下：



```java
@Import(AutoConfigurationPackages.Registrar.class)  //给容器中导入一个组件
public @interface AutoConfigurationPackage {}

//利用Registrar给容器中导入一系列组件
//将指定的一个包下的所有组件导入进来？MainApplication 所在包下组件全部注入IOC。
//Registrar类的作用就是进行批量注册组件，把包下的组件进行批量注册
```



#### 2、@Import(AutoConfigurationImportSelector.class)



> //怎么通过导入AutoConfigurationImportSelector组件，进行批量导入组件的？？？？？
> 1、利用getAutoConfigurationEntry(annotationMetadata);给容器中批量导入一些组件
> 2、调用List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes)获取到所有需要导入到容器中的配置类
> 3、利用工厂加载 Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader)；得到所有的组件
> 4、从META-INF/spring.factories位置来加载一个文件。
>     默认扫描我们当前系统里面所有META-INF/spring.factories位置的文件
>     spring-boot-autoconfigure-2.3.4.RELEASE.jar包里面也有META-INF/spring.factories
>     
> spring-boot-autoconfigure-2.3.4.RELEASE.jar/META-INF/spring.factories文件里面写死了spring-boot一启动就要给容器中加载的所有配置类



## ![img](https://cdn.nlark.com/yuque/0/2020/png/1354552/1602845382065-5c41abf5-ee10-4c93-89e4-2a9b831c3ceb.png)



### 4、按需开启自动配置项



虽然我们127个场景的所有自动配置启动的时候默认全部加载。xxxxAutoConfiguration  按照条件装配规则（@Conditional），最终会按需配置。

修改默认配置

```java
// 给容器中加入了文件上传解析器；
		@Bean
        @ConditionalOnBean(MultipartResolver.class)  //容器中有这个类型组件
        @ConditionalOnMissingBean(name = DispatcherServlet.MULTIPART_RESOLVER_BEAN_NAME) //容器中没有这个名字 multipartResolver 的组件
        public MultipartResolver multipartResolver(MultipartResolver resolver) {
            //给@Bean标注的方法传入了对象参数，这个参数的值就会默认从容器--中找。
            //SpringMVC multipartResolver。防止有些用户配置的文件上传解析器不符合规范
            // Detect if the user has created a MultipartResolver but named it incorrectly
            return resolver;
        }

```



SpringBoot默认会在底层配好所有的组件。但是如果用户自己配置了以用户的优先

```java
@Bean
@ConditionalOnMissingBean
public CharacterEncodingFilter characterEncodingFilter() {
  }
```



**自动配置原理的总结：**



- SpringBoot先加载所有的自动配置类  xxxxxAutoConfiguration
- 每个自动配置类按照条件进行生效，**默认都会绑定配置文件指定的值**。xxxxProperties里面拿。xxxProperties和配置文件进行了绑定

- 生效的配置类就会给容器中装配很多组件
- 只要容器中有这些组件，相当于这些功能就有了

- **定制化配置**
  - **用户直接自己@Bean替换底层的组件**
  - **用户去看这个组件是获取的配置文件什么值就去修改。**（在文档查或者在底层源码查）



**xxxxxAutoConfiguration -----（按需加载）-------> 组件  --->** **xxxxProperties里面拿值  ----> application.properties**



### 5、最佳实践



- **引入场景依赖（jar包）**
- 需要什么样的场景依赖？？这么找？？
  - https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter

- **查看自动配置了哪些（选做：因为是底层原理）**
  - ​	自己分析，引入场景对应的自动配置一般都生效了
  - 配置文件中写  `debug=true` 开启自动配置报告。在执行的时候，就会发现什么组件自动配置了，什么没有配置。Negative（不生效）\Positive（生效）

- **是否需要修改**
  - 参照文档修改配置项
  - https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#common-application-properties
  - 自己分析（看源码自己分析）。xxxxProperties绑定了配置文件的哪些。
  - 自定义加入或者替换组件
  - @Bean、@Component。。。
  - 因为用户有的以用户配置的组件优先
  - 自定义器  **XXXXXCustomizer**；
  - ......





## 3.6、开发小技巧



### 1、Lombok



**简化JavaBean开发**



- 第一步：导入lombok依赖
- 第二步：安装lomabok插件（编译的时候，会帮我们自动生成get、set方法）

- 在设置---插件--搜索lombak插件

- `@Date注解`

- 作用：帮我们生成类中已有属性的get和set方法
- 位置：类上

- `@ToString注解`

- 作用：在编译类的时候，帮我们生成tostring方法
- 位置：类上

- `@AllArgsConstructor注解`

- 作用：全参构造方法
- 位置：类上

- `@NoArgsConstructor注解`

- 作用：无参构造方法
- 位置：类上

- `@EqualsAndHashCode注解`

- 作用：通过属性，重写HashCode方法
- 位置：类上



```java
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

idea中搜索安装lombok插件
===============================简化JavaBean开发===================================
@NoArgsConstructor
//@AllArgsConstructor
@Data
@ToString
@EqualsAndHashCode
public class User {

    private String name;
    private Integer age;

    private Pet pet;

    public User(String name,Integer age){
        this.name = name;
        this.age = age;
    }
}

```

### 2. @Slf4j日志

```java
================================简化日志开发===================================
@Slf4j  //日志记录器，会在该类中自动注入一个 log属性，可以调用log属性的方法
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String handle01(@RequestParam("name") String name){
        log.info("请求进来了....");  // 在控制台输出日志信息
        return "Hello, Spring Boot 2!"+"你好："+name;
    }
}
```

### 3、dev-tools

- 作用：热更新
- （项目更新，不需要重新启动就可以重新编译（ctrl+F9））



```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
```



项目或者页面修改以后：Ctrl+F9；





# 4. 配置文件

spring使用properties配置文件

springboot推荐使用`yaml`

非常适合用来做**以数据为中心的配置文件**

## 1、基本语法

> - key: value；kv之间有空格
> - 大小写敏感
>
> - 使用缩进表示层级关系
> - 缩进**不允许使用tab**，只允许空格
>
> - 缩进的空格数不重要，只要相同层级的元素左对齐即可
> - '#'表示注释
>
> - **字符串无需加引号**，如果要加，' '与" "表示字符串内容 会被 转义/不转义

## 2、数据类型



- 字面量：单个的、不可再分的值。date、boolean、string、number、null

```yaml
k: v
```

- 对象：键值对的集合。map、hash、set、object 

```yaml
行内写法：  k: {k1:v1,k2:v2,k3:v3}
#或
k: 
  k1: v1
  k2: v2
  k3: v3
```

- 数组：一组按次序排列的值。array、list、queue

```yaml
行内写法：  k: [v1,v2,v3]
#或者
k:
 - v1
 - v2
 - v3
```

## 3、示例

```java
@Data
public class Person {
	
	private String userName;
	private Boolean boss;
	private Date birth;
	private Integer age;
	private Pet pet;
	private String[] interests;
	private List<String> animal;
	private Map<String, Object> score;
	private Set<Double> salarys;
	private Map<String, List<Pet>> allPets;
}

@Data
public class Pet {
	private String name;
	private Double weight;
}
```



```yaml
# yaml表示以上对象
person:
  userName: zhangsan
  boss: false
  birth: 2019/12/12 20:12:33
  age: 18
  pet: 
    name: tomcat
    weight: 23.4
  interests: [篮球,游泳]
  animal: 
    - jerry
    - mario
  score:
    english: 
      first: 30
      second: 40
      third: 50
    math: [131,140,148]
    chinese: {first: 128,second: 136}
  salarys: [3999,4999.98,5999.99]
  allPets:
    sick:
      - {name: tom}
      - {name: jerry,weight: 47}
    health: [{name: mario,weight: 47}]
```



## 4、配置提示

自定义的类和配置文件绑定一般没有提示。

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>


 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.springframework.boot</groupId>
                            <artifactId>spring-boot-configuration-processor</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

# 5. Web开发

##  5.1 SpringMVC自动配置

 Spring Boot 为 Spring MVC 提供了自动配置，适用于大多数应用程序。

自动配置在 Spring 的默认值之上添加了以下功能：

- 包括`ContentNegotiatingViewResolver`和`BeanNameViewResolver`bean。
- 支持提供静态资源，包括对` WebJars `的支持（[本文档稍后介绍](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.static-content)）。
- 自动注册`Converter`，`GenericConverter`和`Formatter`bean类。
- 支持`HttpMessageConverters`（[在本文档后面介绍](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.message-converters)）。
- 自动注册`MessageCodesResolver`（[在本文档后面介绍](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.message-codes)）。
- 静态`index.html`支持。
- `ConfigurableWebBindingInitializer`bean 的自动使用（[本文档稍后介绍](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.binding-initializer)）。

如果您想保留那些 `Spring Boot MVC` 自定义并进行更多[MVC 自定义](https://docs.spring.io/spring-framework/docs/5.3.8/reference/html/web.html#mvc)（拦截器、格式化程序、视图控制器和其他功能），您可以添加自己`@Configuration`的类型类，`WebMvcConfigurer`但**不添加** `@EnableWebMvc`.

如果你想提供的定制情况`RequestMappingHandlerMapping`，`RequestMappingHandlerAdapter`或者`ExceptionHandlerExceptionResolver`，仍然保持springbootMVC自定义，你可以声明类型的豆`WebMvcRegistrations`，并用它来提供这些组件的定制实例。

如果你想利用Spring MVC中的完全控制，你可以添加自己的`@Configuration`注解为`@EnableWebMvc`，或者添加自己的`@Configuration`-annotated`DelegatingWebMvcConfiguration`中的Javadoc中所述`@EnableWebMvc`。



> 1. **不用@EnableWebMvc注解。使用** `@Configuration` **+** `WebMvcConfigurer` **自定义规则**
>
> 2. **声明** `WebMvcRegistrations` **改变默认底层组件**
>
> 3. **使用** `@EnableWebMvc+@Configuration+DelegatingWebMvcConfiguration` 全面接`SpringMVC`

## 5.2 基本功能使用

### 5.2.1 静态资源目录

静态资源一般指：css, js, font, img

只要静态资源放在类路径下：  `/static` (or `/public` or `/resources` or `/META-INF/resources`

访问 ： 当前项目根路径/ + 静态资源名 `http://localhost:8080/1.png`

原理： 静态映射/**。底层代码在`spring.factories`文件-->`WebMvcAutoConfiguration`类--》`WebMvcAutoConfigurationAdapter`方法--> `webProperties`类 -->` CLASSPATH_RESOURCE_LOCATIONS`静态数组

1. 请求进来，先去找Controller看能不能处理。

2. 不能处理的所有请求又都交给静态资源处理器。

3. 静态资源也找不到则响应404页面

```yaml
spring:
  web:
    resources:
      static-locations: [classpath:/haha/, classpath:/gaga/] # 修改静态资源路径，原路径会失效
```

### 5.2.2、静态资源访问前缀

默认无前缀,添加前缀后所有访问需要：`localhost:/res/{path}`

```yaml
spring:
  mvc:
    static-path-pattern: /res/**
```

### 5.2.3、webjar

自动映射 /[webjars](http://localhost:8080/webjars/jquery/3.5.1/jquery.js)/**

https://www.webjars.org/

```xml
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.5.1</version>
        </dependency>
```

访问地址：[http://localhost:8080/webjars/**jquery/3.5.1/jquery.js**](http://localhost:8080/webjars/jquery/3.5.1/jquery.js)   后面地址要按照依赖里面的包路径

### 5.2.4 首页

- 静态资源路径下  index.html
  - 可以配置静态资源路径,需要将`index.html`文件放在静态资源包下才能被访问
  - 但是不可以配置静态资源的访问前缀。否则导致 index.html不能被默认访问

```yaml
spring:
#  mvc:
#    static-path-pattern: /res/**   这个会导致welcome page功能失效

  resources:
    static-locations: [classpath:/haha/]
```

- controller能处理/index

### 5.2.5、自定义 `Favicon`

`favicon.ico` 放在静态资源目录下即可。

```yaml
spring:
#  mvc:
#    static-path-pattern: /res/**   这个会导致 Favicon 功能失效
```

### 5.2.6、静态资源配置原理

底层代码在`spring.factories`文件-->`WebMvcAutoConfiguration`类--》`WebMvcAutoConfigurationAdapter`方法--> `webProperties`类 

- `SpringBoot`启动默认加载  `xxxAutoConfiguration` 类（自动配置类）
- `SpringMVC`功能的自动配置类 `WebMvcAutoConfiguration`，生效

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {}
```

- 给容器中配了什么。

```java
	@Configuration(proxyBeanMethods = false)
	@Import(EnableWebMvcConfiguration.class)
	@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
	@Order(0)
	public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {}
```

- 配置文件的相关属性和xxx进行了绑定。`WebMvcProperties`==**spring.mvc**、ResourceProperties==**spring.resources**

#### 1、配置类只有一个有参构造器

```java
	//有参构造器所有参数的值都会从容器中确定
//ResourceProperties resourceProperties；获取和spring.resources绑定的所有的值的对象
//WebMvcProperties mvcProperties 获取和spring.mvc绑定的所有的值的对象
//ListableBeanFactory beanFactory Spring的beanFactory
//HttpMessageConverters 找到所有的HttpMessageConverters
//ResourceHandlerRegistrationCustomizer 找到 资源处理器的自定义器。=========
//DispatcherServletPath  
//ServletRegistrationBean   给应用注册Servlet、Filter....
	public WebMvcAutoConfigurationAdapter(
        ResourceProperties resourceProperties, WebMvcProperties mvcProperties,
				ListableBeanFactory beanFactory, ObjectProvider<HttpMessageConverters> messageConvertersProvider,
				ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider,
				ObjectProvider<DispatcherServletPath> dispatcherServletPath,
				ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
			this.resourceProperties = resourceProperties;
			this.mvcProperties = mvcProperties;
			this.beanFactory = beanFactory;
			this.messageConvertersProvider = messageConvertersProvider;
			this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
			this.dispatcherServletPath = dispatcherServletPath;
			this.servletRegistrations = servletRegistrations;
		}
```





#### 2、资源处理的默认规则

```java
		@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
			//webjars的规则
            if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
            
            //
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
						.addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
		}
```

```yaml
spring:
#  mvc:
#    static-path-pattern: /res/**
  resources:
    add-mappings: false   禁用所有静态资源规则
```

```java    
	@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {
	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
			"classpath:/resources/", "classpath:/static/", "classpath:/public/" };

	/**
	 * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
	 * /resources/, /static/, /public/].
	 */
	private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
```



#### 3、欢迎页的处理规则

​	`HandlerMapping`：处理器映射。保存了每一个`Handler`能处理哪些请求。	

```java
	@Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
				FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
			WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
					new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
					this.mvcProperties.getStaticPathPattern());
			welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
			welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
			return welcomePageHandlerMapping;
		}
	
	WelcomePageHandlerMapping(TemplateAvailabilityProviders templateAvailabilityProviders,
			ApplicationContext applicationContext, Optional<Resource> welcomePage, String staticPathPattern) {
		if (welcomePage.isPresent() && "/**".equals(staticPathPattern)) {
	        //要用欢迎页功能，必须是/**
			logger.info("Adding welcome page: " + welcomePage.get());
			setRootViewName("forward:index.html");
		}
		else if (welcomeTemplateExists(templateAvailabilityProviders, applicationContext)) {
	        // 调用Controller  /index
			logger.info("Adding welcome page template: index");
			setRootViewName("index");
		}
	}
```



## 5.3 请求参数处理

### 5.3.1 请求映射

**rest使用与原理**

- @xxxMapping；（Get）
- Rest风格支持（*使用**HTTP**请求方式动词来表示对资源的操作*）
  - *以前：**/getUser*  *获取用户*    */deleteUser* *删除用户*   */editUser*  *修改用户*      */saveUser* *保存用户*
  - *现在： /user*    *GET-**获取用户*    *DELETE-**删除用户*     *PUT-**修改用户*      *POST-**保存用户*
  - 核心Filter；HiddenHttpMethodFilter
  - 用法： 表单method=post，隐藏域 _method=put
  - **SpringBoot中手动开启**
  - 扩展：如何把_method 这个名字换成我们自己喜欢的。

```java
    @RequestMapping(value = "/user",method = RequestMethod.GET)
	//@GetMapping("/user")
    public String getUser(){
        return "GET-张三";
    }

    @RequestMapping(value = "/user",method = RequestMethod.POST)
	//@PostMapping("/user")
    public String saveUser(){
        return "POST-张三";
    }


    @RequestMapping(value = "/user",method = RequestMethod.PUT)
	//@PutMapping("/user")
    public String putUser(){
        return "PUT-张三";
    }

    @RequestMapping(value = "/user",method = RequestMethod.DELETE)
	//@DeleteMapping("/user")
    public String deleteUser(){
        return "DELETE-张三";
    }

```

开启方式一：

```yaml
spring:
  mvc:
    hiddenmethod:
      filter:
        enabled: true   #开启页面表单的Rest功能
```

方式二：

```java
@Configuration(proxyBeanMethods = false)
public class myconfig {
    
	@Bean
	@ConditionalOnMissingBean(HiddenHttpMethodFilter.class)
	@ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled", matchIfMissing = false)
	public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
		return new OrderedHiddenHttpMethodFilter();
	}

	//方式三：自定义filter
    @Bean
    public HiddenHttpMethodFilter hiddenHttpMethodFilter(){
        HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
        methodFilter.setMethodParam("_m");
        return methodFilter;
    }
}    
```



Rest原理（表单提交要使用REST的时候）

> - 表单提交会带上**_method=PUT**
> - **请求过来被**HiddenHttpMethodFilter拦截
>
> - 请求是否正常，并且是POST
>
> - 获取到**_method**的值。
> - 兼容以下请求；**PUT**.**DELETE**.**PATCH**
>
> - **原生request（post），包装模式requesWrapper重写了getMethod方法，返回的是传入的值。**
> - 过滤器链放行的时候用wrapper。以后的方法调用getMethod是调用requesWrapper的。

**Rest使用客户端工具，**

- 如PostMan直接发送Put、delete等方式请求，无需Filter。

**请求映射原理**

![img](https://cdn.nlark.com/yuque/0/2020/png/1354552/1603181171918-b8acfb93-4914-4208-9943-b37610e93864.png)

SpringMVC功能分析都从 org.springframework.web.servlet.DispatcherServlet-》doDispatch（）



```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// 找到当前请求使用哪个Handler（Controller的方法）处理
				mappedHandler = getHandler(processedRequest);
                
                //HandlerMapping：处理器映射。/xxx->>xxxx
```

![img](https://cdn.nlark.com/yuque/0/2020/png/1354552/1603181460034-ba25f3c0-9cfd-4432-8949-3d1dd88d8b12.png)

**RequestMappingHandlerMapping**：保存了所有@RequestMapping 和handler的映射规则。

![img](https://cdn.nlark.com/yuque/0/2020/png/1354552/1603181662070-9e526de8-fd78-4a02-9410-728f059d6aef.png)

所有的请求映射都在HandlerMapping中。



- SpringBoot自动配置欢迎页的 WelcomePageHandlerMapping 。访问 /能访问到index.html；
- SpringBoot自动配置了默认 的 RequestMappingHandlerMapping

- 请求进来，挨个尝试所有的HandlerMapping看是否有请求信息。

- 如果有就找到这个请求对应的handler
- 如果没有就是下一个 HandlerMapping

- 我们需要一些自定义的映射处理，我们也可以自己给容器中放**HandlerMapping**。自定义 **HandlerMapping**

```java
	protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		if (this.handlerMappings != null) {
			for (HandlerMapping mapping : this.handlerMappings) {
				HandlerExecutionChain handler = mapping.getHandler(request);
				if (handler != null) {
					return handler;
				}
			}
		}
		return null;
	}
```



### 5.3.2 普通参数和基本注解

`@PathVariable` 路径变量、`@RequestHeader` 获取请求头、`@RequestParam `获取请求参数、`@RequestBody `获取请求体

`@CookieValue` 获取cookie值、`@RequestAttribute` 获取请求域中的值、`ModelAttribute ` 、 `@MatrixVariable` 矩阵变量



请求的测试地址：`/car/1/owner/zs?age=10&inters=aa&inters=bb`



```java
@RestController
public class ParameterTestController {

    //  /car/1/owner/zs?age=10&inters=aa&inters=bb
    @GetMapping("/car/{id}/owner/{username}")  //中括号中表示要参数的值
    public Map<String,Object> getCar(@PathVariable("id") Integer id,  //@PathVariable可以获取路径上的参数的值做为参数
                                     @PathVariable("username") String name,
                                     @PathVariable Map<String,String> pv, //@PathVariable注解可以把所有的参数值，放入一个Map<String,String>中
                            		 @RequestHeader(value = "User-Agent",required = false) String userAgent,//获取User-Agent请求头的值并且不是必须有的
                                     @RequestHeader Map<String,String> header, //获取所有的请求头，放入一个Map<String,String>中
                                     @RequestParam("age") Integer age, //获取请求参数中的值
                                     @RequestParam("inters") List<String> inters,//请求参数为inters的值放入集合
                                     @RequestParam Map<String,String> params,//所有请求参数值放入集合
                                     @CookieValue("_ga") String _ga,//获取值为_ga的cookie的值(单独某一个Cookie的值)
                                     @CookieValue("_ga") Cookie cookie){ //拿到某个Cookie的值


        Map<String,Object> map = new HashMap<>();

        map.put("id",id);
        map.put("name",name);
        map.put("pv",pv);
        map.put("userAgent",userAgent);
        map.put("headers",header);
        map.put("age",age);
        map.put("inters",inters);
        map.put("params",params);
        map.put("_ga",_ga);
        System.out.println(cookie.getName()+"===>"+cookie.getValue());
        return map;
    }


    //获取请求体（使用form发生post请求)
    @PostMapping("/save")
    public Map postMethod(@RequestBody String content){ //把请求体的数据放到参数中
        Map<String,Object> map = new HashMap<>();
        map.put("content",content);
        return map;
    }

    
    //获取请求参数
    @GetMapping("/goto")
	public String goToPage(HttpServletRequest request){
		request.setAttribute("msg","成功");
		request.setAttribute("code",200);
		//跳转到 /success请求
		return "forward:/success";
	}


	@ResponseBody
	@GetMapping("/success")
	public Map<String,Object> success(@RequestAttribute("msg") String msg ,  //  使用@RequestAttribute注解取出请求域中的值
						  @RequestAttribute("code") Integer code,
						  HttpServletRequest request){    //这个request和goToPage方法中的request是相同的，使用api得到请求参数

		Map<String,Object> map = new HashMap<>();
		map.put("msg",msg);
		map.put("code",request.getAttribute("code"));
		return map;
	}
    
    

```



@MatrixVariable

```java
	//矩阵变量
    //1、语法： 请求路径：/cars/sell;low=34;brand=byd,audi,yd  带分号的为矩阵变量，一个可以可以对应一个或多个值
    //2、SpringBoot默认是禁用了矩阵变量的功能
    //      手动开启：
    //			原理。对于路径的处理。UrlPathHelper进行解析。removeSemicolonContent属性（移除分号内容）支持矩阵变量的
    //           怎么开启？？
    //				方法一：
    //					自定义一个 接口的实现 组件放到容器，设置其UrlPathHelper对象中的removeSemicolonContent属性
    //				    @Configuration(proxyBeanMethods = false)
    //					public class WebConfig {
    //						@Bean
    //						public WebMvcConfigurer webMvcConfigurer() {
    //							return new WebMvcConfigurer(
    //							@Override
    //							public void configurePathMatch(PathMatchConfigurer configurer) {
    //								UrlPathHelper urlPathHelper=new UrlPathHelper();
    //								urlPathHelper.setRemoveSemicolonContent(false);
    //								configurer.setUrlPathHelper(urlPathHelper);
    //							}
    //							);
    //						}
    //					}
    //				方法二：
    //					直接实现该接口
    // 					@Configuration
    //                       public class WebConfig implements WebMvcConfigurer {
    //                            @Override
    //                            public void configurePathMatch(PathMatchConfigurer configurer) {
    //                                UrlPathHelper urlPathHelper=new UrlPathHelper();
    //                                urlPathHelper.setRemoveSemicolonContent(false);
    //                               configurer.setUrlPathHelper(urlPathHelper);
    //                            }
    //                        }
    //3、矩阵变量必须有url路径变量才能被解析
    
    
    //   /cars/sell;low=34;brand=byd,audi,yd 
    @GetMapping("/cars/{path}")
    public Map carsSell(@MatrixVariable("low") Integer low,
                        @MatrixVariable("brand") List<String> brand,
                        @PathVariable("path") String path){
        Map<String,Object> map = new HashMap<>();
        map.put("low",low);
        map.put("brand",brand);
        map.put("path",path);
        return map;
    }

    // /boss/1;age=20/2;age=10
    //  对于不同路径下的的age值怎么获取？？ 通过路劲进行指定
    @GetMapping("/boss/{bossId}/{empId}")
    public Map boss(@MatrixVariable(value = "age",pathVar = "bossId") Integer bossAge,  //通过pathVar来指定是哪个路径下的value值
                    @MatrixVariable(value = "age",pathVar = "empId") Integer empAge){
        Map<String,Object> map = new HashMap<>();

        map.put("bossAge",bossAge);
        map.put("empAge",empAge);
        return map;
    }

}
```



**复杂参数：**

**Map**、**Model（map、model里面的数据会被放在request的请求域  request.setAttribute）、**

Errors/BindingResult、**RedirectAttributes（ 重定向携带数据）**、**ServletResponse（response）**、SessionStatus、UriComponentsBuilder、ServletUriComponentsBuilder

```java
@Controller
public class RequestController {

    @GetMapping("/req")
    public String req(Map<String, Object> map,
                      Model model,
                      HttpServletRequest request,
                      HttpServletResponse response){

        map.put("hello", "jnu");
        model.addAttribute("ni", "hao");
        request.setAttribute("my name", "haha");

        Cookie cookie = new Cookie("c1", "v1");
        response.addCookie(cookie);

        return "forward:/success";
        //当请求req时会跳转到success页面并携带request的参数，如map/model/request
    }

    @ResponseBody
    @GetMapping("/success")
    public Map success(HttpServletRequest request){
        Map<String, Object> map = new HashMap<>();
        Object hello = request.getAttribute("hello");
        Object ni = request.getAttribute("ni");
        Object my_name = request.getAttribute("my name");
        map.put("hello", hello);
        map.put("ni", ni);
        map.put("my_name", my_name);

        return map;
    }

}
```







## 5.4 视图解析

**视图解析原理流程**

> 1、目标方法处理的过程中，所有数据都会被放在 **ModelAndViewContainer 里面。包括数据和视图地址**
>
> **2、方法的参数是一个自定义类型对象（从请求参数中确定的），把他重新放在** **ModelAndViewContainer** 
>
> 3、任何目标方法执行完成以后都会返回 ModelAndView（数据和视图地址）。
>
> 4、processDispatchResult  处理派发结果（页面改如何响应）
>
> - 1、**render**(**mv**, request, response); 进行页面渲染逻辑
>   - 1、根据方法的String返回值得到 **View** 对象【定义了页面的渲染逻辑】
>     - 1、所有的视图解析器尝试是否能根据当前返回值得到**View**对象
>     - 2、得到了  **redirect:/main.html** --> Thymeleaf new **RedirectView**()
>     - 3、ContentNegotiationViewResolver 里面包含了下面所有的视图解析器，内部还是利用下面所有视图解析器得到视图对象。
>     - 4、view.render(mv.getModelInternal(), request, response);   视图对象调用自定义的render进行页面渲染工作
>     - **RedirectView 如何渲染【重定向到一个页面】**
>       - **1、获取目标url地址**
>       - *2*、*response.sendRedirect(encodedURL);*

**视图解析：**

- **返回值以 forward: 开始： new InternalResourceView(forwardUrl); -->  转发****request.getRequestDispatcher(path).forward(request, response);** 
- **返回值以** **redirect: 开始：** **new RedirectView() --》 render就是重定向** 

- **返回值是普通字符串： new ThymeleafView（）--->** 

## 5.5 模板引擎

### **1、thymeleaf简介**

[具体详情地址](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#standard-expression-syntax)

Thymeleaf is a modern server-side Java template engine for both web and standalone environments, capable of processing HTML, XML, JavaScript, CSS and even plain text.

**现代化、服务端Java模板引擎**



### **2、基本语法**

#### 1、表达式

| 表达式名字 | 语法   | 用途                               |
| ---------- | ------ | ---------------------------------- |
| 变量取值   | ${...} | 获取请求域、session域、对象等值    |
| 选择变量   | *{...} | 获取上下文对象值                   |
| 消息       | #{...} | 获取国际化等值                     |
| 链接       | @{...} | 生成链接                           |
| 片段表达式 | ~{...} | jsp:include 作用，引入公共页面片段 |



#### 2、**字面量**

文本值: **'one text'** **,** **'Another one!'** **,…**数字: **0** **,** **34** **,** **3.0** **,** **12.3** **,…**布尔值: **true** **,** **false**

空值: **null**

变量： one，two，.... 变量不能有空格

```html
 <!--  把文本内容换成 请求域中的message信息 -->
<!--  下面几种方式都可以实现  -->
 <p th:text="${message}">Welcome to BeiJing!</p>
 <p>[[${message}}]]</p>
 <!--取出请求域中user对象的role属性的RoleName属性的值-->
 <p th:text="${user.role.RoleName}"><p>

 <!--文本-->
 <p th:text="'Welcome to BeiJing!'"></p>
 <p th:text="'\'Welcome to BeiJing!\''"></p>

 <!-- 数字字面值-->
 <!-- 2017 -->
 <p th:text="2017"></p>
 <!-- 2018 -->
 <p th:text="2017 + 1"></p>

 <!-- 布尔字面值   -->
 <!-- false -->
 <p th:text="1 > 2"></p>
 <!-- 否 -->
 <p th:text="1 > 2 ? '是' : '否'"></p>
 
 <!--空字面值-->
 <!-- false -->
 <p th:text="${user == null}"></p>

 <!--字符串连接-->
 <!-- Welcome to BeiJing! -->
 <p th:text="'Welcome to ' + ${location} + '!'"></p>

 <!-- 字面值替换-->
 <!--符号||可以用来将字面值和表达式包裹起来，这样就能方便的替换变量的值，而不需要使用+连接符：-->
 <!-- Welcome to BeiJing! -->
 <p th:text="|Welcome to ${location}!|"></p>

 <!--条件运算--三目运算符: (if) ? (then) : (else) -->
 <p th:text="${user.online ? '在线' : '离线'}"></p>
 <p th:text="${user.online ? (user.vip ? 'VIP用户在线' : '普通用户在线') : '离线'}"></p>

 <!-- 二元运算符：(value) ?: (defaultValue) -->
 <!--value非空（null）即真，条件为真时输出value，否则输出defaultValue-->
 <!-- 你还没有登录，请先登录 -->
 <p th:text="${token} ?: '你还没有登录，请先登录'"></p>
 <!-- fanlychie@gmail.com -->
 <p th:text="${user.email} ?: '你还没有绑定邮箱'"></p>


 <!-- 布尔运算-->
 <p th:text="${user.online and user.vip}"></p>
 <p th:text="${user.online or user.vip}"></p>
 <p th:text="${!user.online}"></p>
 <p th:text="${not user.online}"></p>

 <!--比较和相等-->
 <p th:text="${user.age < 60}"></p>
 <p th:text="${user.age <= 60}"></p>
 <p th:text="${user.age > 18}"></p>
 <p th:text="${user.age >= 18}"></p>
 <p th:text="${user.age == 18}"></p>
 <p th:text="${user.age != 18}"></p>

 <!--无操作符-->
 <!--当模板运行在服务器端时，Thymeleaf 会解析th:*属性的具体值替换标签体的内容。无操作符（_）则允许你使用原型标签体的内容作为默认值：-->
 <!-- 你还没有登录，请先登录 -->
 <p th:text="${token} ?: _">你还没有登录，请先登录</p>
```



#### 3. 文本内容的替换

- `th:text`
  - 作为静态文件直接运行时，浏览器会自动忽略它不能识别的`th:text`属性

  - 当它作为模板文件运行在服务器端时，`th:text`属性的具体值将会替换文本内容

  - 默认会对含有 HTML 标签的内容进行字符转义

  - `[[]]`相当于`th:text`，对含有 HTML 标签的内容自动进行字符转义。

    `<p>The message is : [[${htmlContent}]]</p>`

    

- `th:utext`
  - 则不会对含有 HTML 标签的内容进行字符转义；

  - `[()]`相当于`th:utext`，对含有 HTML 标签的内容不进行字符转义。

    `<p>The message is : [(${htmlContent})]</p>`

```html
<p th:text="${message}">Welcome to BeiJing!</p>
<!--    假设：message = "<b>Welcome to BeiJing!</b>"-->
<p th:text="${message}"></p>   <!--  Welcome to BeiJing! -->
<p th:utext="${message}"></p>   <!--<b>Welcome to BeiJing!</b>-->
```



#### 4. URL的地址语法解析

- 使用 `@{...}`
- 使用 th:属性 指定我们的URL ，当然在URL路径上可以采用变量的方式设置动态的URL地址

```html
<!-- 全路径的写法    -->
<a href="www.taobao.com" th:href="@{https://www.baidu.com/}">百度</a>
<!-- 相对路径的写法
    会根据工程的，自动的加上前缀路径 得到结果：http://localhost:8080/text2
-->
<a href="www.taobao.com" th:href="@{/text2}">Controller控制器</a>
```



#### 5. 变量中的实用对象

- ctx 、 vars 、 locale 、request 、response 、servletContext 、dates 等等内置对象 可像在jsp页面的内置对象一样使用
- 使用方法：
  - 使用  `#` 来引用 内置对象

```html
<p th:text="${#dates.format(now,'YYYY-MM-dd'}">
<!-- 
	${#request.getAttribute('foo')}
	${#request.getParameter('foo')}
	${#session.getAttribute('foo')}
	${#session.id}
	${#dates.arrayFormat(datesArray)}
	${#dates.day(date)} 
-->
```



#### 6. **迭代**

- 遍历（迭代）的语法`th:each="自定义的元素变量名称 : ${集合变量名称}"`：

```html
<table>
    <tr>
        <th>姓名</th>
        <th>年龄</th>
    </tr>

    <tr th:each="user: ${users}">
        <td th:text="${user.username}"></td>
        <td th:text="${user.age}"></td>
    </tr>
</table>
```



#### 7. **抽取公共代码**

- 对应 使用  `th:fragment="copy"`  方式抽取

- 在公共代码的最外层标签添加 th:fragment="copy"
- 怎么引用？？

```
<div th:insert="~{公共页面名称 :: copy}"></div>
<div th:insert="公共页面名称 :: copy"></div>
```

- `th:insert`是最简单的：它将简单地插入指定的片段作为其主机标记的主体。
- `th:replace`其实*取代*带有指定片段的主机标记。

- `th:include`类似于`th:insert`，但它不是插入片段，而是插入*内容*这个碎片。
- ![img](https://cdn.nlark.com/yuque/0/2021/png/12563077/1615810666986-4b783210-9cd4-4bc4-b224-743bb57d0e2d.png)

- 对于使用 `id选择器` 方式抽取

- 在公共代码的最外层标签添加  id
- 怎么引用？？(比起上面的方式，添加 一个 #)

```
<div th:insert="~{公共页面名称 :: #id选择器名称}"></div>
```

- 和上面的公共代码的引用是一样的，`添加 #`

#### 8. 条件判断



- 条件判断语句有三种，分别是：`th:if`、`th:unless`、`th:swith`
- `th:if`

```
<a  th:if="${user != null}">我的订单</a>
```

- `th:unless`

- `th:unless`与`th:if`判断恰好相反，当表达式的评估结果为假时则显示内容，否则不显示：

```
<a  th:unless="${user == null}">我的订单</a>
```

- `th:swith`

- 多路选择语句，它需要搭配`th:case`来使用：

```
<div th:switch="${user.role}">
    <p th:case="admin">管理员</p>
    <p th:case="user">普通用户</p>
</div>
```



更多更详细的可以参考`官网学习`  或者  `https://blog.csdn.net/qq_33034955/article/details/108474232`





### 3.**使用示例**

1. 先在pom文件中添加依赖`thymeleaf`

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```

2. 在`control`中写入相关变量控制

```java
@Controller
public class ViewController {

    @GetMapping("/login")
    public String login(Model model){
        model.addAttribute("msg", "hello");
        model.addAttribute("link", "www.baidu.com");
        return "login";
    }

}
```

3. 必须要在`templates`文件夹下添加相关的`HTML`文件,注意：开头必须添加`thymeleaf`的模板`xmlns:th="http://www.thymeleaf.org"`

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
   <!--  把文本内容换成 请求域中的message信息 -->
   <!--  下面几种方式都可以实现  -->
    <p th:text="${message}">Welcome to BeiJing!</p>
    <p>[[${message}}]]</p>

<h2>
    <a  href="www.baidu.com" th:href="${link}" >百度1</a><br>
    <a href="www.baidu.com" th:href="@{/link}">百度1</a>
</h2>
</body>
</html>
```



## 5.6 文件上传

### 1、页面表单

```html
<form method="post" action="/upload" enctype="multipart/form-data">
    <input type="file" name="file"><br>
    <input type="submit" value="提交">
</form>
```



### 2、文件上传代码

```java
    /**
     * MultipartFile 自动封装上传过来的文件
     * @param email
     * @param username
     * @param headerImg
     * @param photos
     * @return
     */
    @PostMapping("/upload")
    public String upload(@RequestParam("email") String email,
                         @RequestParam("username") String username,
                         @RequestPart("headerImg") MultipartFile headerImg,
                         @RequestPart("photos") MultipartFile[] photos) throws IOException {

        log.info("上传的信息：email={}，username={}，headerImg={}，photos={}",
                email,username,headerImg.getSize(),photos.length);

        if(!headerImg.isEmpty()){
            //保存到文件服务器，OSS服务器
            String originalFilename = headerImg.getOriginalFilename();
            headerImg.transferTo(new File("H:\\cache\\"+originalFilename));
        }

        if(photos.length > 0){
            for (MultipartFile photo : photos) {
                if(!photo.isEmpty()){
                    String originalFilename = photo.getOriginalFilename();
                    photo.transferTo(new File("H:\\cache\\"+originalFilename));
                }
            }
        }


        return "main";
    }
```

### 3. 自动配置原理

**文件上传自动配置类-MultipartAutoConfiguration-****MultipartProperties**

- 自动配置好了 **StandardServletMultipartResolver   【文件上传解析器】**
- **原理步骤**

- **1、请求进来使用文件上传解析器判断（**isMultipart**）并封装（**resolveMultipart，**返回**MultipartHttpServletRequest**）文件上传请求**
- **2、参数解析器来解析请求中的文件内容封装成MultipartFile**

- **3、将request中文件信息封装为一个Map；**MultiValueMap<String, MultipartFile>

**FileCopyUtils**。实现文件流的拷贝

```
    @PostMapping("/upload")
    public String upload(@RequestParam("email") String email,
                         @RequestParam("username") String username,
                         @RequestPart("headerImg") MultipartFile headerImg,
                         @RequestPart("photos") MultipartFile[] photos)
```

![img](https://cdn.nlark.com/yuque/0/2020/png/1354552/1605847414866-32b6cc9c-5191-4052-92eb-069d652dfbf9.png)



















