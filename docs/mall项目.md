

# 使用问题

### 问题1

使用mybatis的generator时：`The specified target project directory mall\src\main\java does not exist`

将`generatorConfig.xml`直接修改为绝对路径即可

```xml
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.ahang.mbg.mapper"
                             targetProject="C:\Users\Ahang\IdeaProjects\mall\src\main\java"/>
```



### 问题2

使用mybatis的generator时`STACKTRACE:javax.net.ssl.SSLException: closing inbound before receiving peer's close_notify`

在`genetator.properties`后面追加`useSSL=false`即可`jdbc.connectionURL=jdbc:mysql://localhost:3306/mall?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai&useSSL=false`



### 问题3

在`PmsBrandServiceImpl`中IDEA出现brandMapper飘红

```java
@Autowired
private PmsBrandMapper brandMapper;
```

替换掉`@Autowired`为`@Resource`



## 问题4

在实现controller接口时里面的CommonResult等common里面的类没有定义



## 问题5

# [MyBatis Generator使用过程中踩过的一个坑](http://www.macrozheng.com/#/technology/mybatis_mapper?id=mybatis-generator%e4%bd%bf%e7%94%a8%e8%bf%87%e7%a8%8b%e4%b8%ad%e8%b8%a9%e8%bf%87%e7%9a%84%e4%b8%80%e4%b8%aa%e5%9d%91)

原文故意没有修改这个坑，我原以为是作者已经有了其他的解决办法按照文档一步一步就好了，没想到源文档是有问题的，需要我们后期通过这个方法去修改。不修改就得手动每次自己删除xml文件



## 问题6

使用了Swagger-UI后有页面显示，但是没有Controller的信息

修改`Swagger2Config`中`apis(RequestHandlerSelectors.basePackage("com.macro.mall.tiny.controller")`修改为自己controller的位置

## 问题7

在spring节点下添加Redis的配置中`redis`是属于`spring`下的，不是直接定格放置的











