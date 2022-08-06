
# postman
​                [官方文档](https://learning.postman.com/docs/writing-scripts/script-references/postman-sandbox-api-reference/)

[postman 的基础使用篇(一)](https://segmentfault.com/a/1190000011991458)
[postman发送请求使用篇(二)](https://segmentfault.com/a/1190000011991614)
[postman响应使用篇(三)](https://segmentfault.com/a/1190000012007906)
[postman的代理使用篇(四)](https://segmentfault.com/a/1190000012024844)
[postman认证使用篇(五)](https://segmentfault.com/a/1190000012056247)
[postman 变量使用篇(六)](https://segmentfault.com/a/1190000012077563)



## 获取设置全局变量

```js
// 从响应头中获取数据
        var Cookie = postman.getResponseHeader("Cookie");
// 设置变量到全局环境中，方便后面调用
// 调用方式{{COOKIE}}
        pm.globals.set("COOKIE", Cookie);

// 获取响应的某个Cookie
        var jsessionid = postman.getResponseCookie("JSESSIONID").value;


// 从响应体中获取数据，然后将数据转化为json对象
// 例如该响应体如下：
        {
    "\result\": "2",
    "forceModifyPassword": false,
    "url": "https://login.gamma.welink.huawei.com/wbackend/"
        }

        var result = JSON.parse(responseBody);
        var result = pm.response.json();
// 打印到平台上
        console.log(result.url) // 获取该对象的url

// 从响应体中用正则方法匹配：将需要匹配的位置用(.*?)替换，
        var result = responseBody.match(new RegExp('"url":"(.*?)"'));
        console.log(result[1]);

// 正则匹配时如果遇到反斜杠 \ ，那么需要加个反斜杠 \\
        var result = responseBody.match(new RegExp('"url":"(.*?)"'));
        console.log(result[1]);

```












# @order
​        使用order属性，设置该类在spring容器中的加载顺序

​        例如有三个类：`Order1,Order2,Order3`，其中Order1类如下：

```java
        @Component //把类交给spring容器管理
        @Order(1)  //使用order属性，设置该类在spring容器中的加载顺序
        public class Order1{
            private final int ORDERED = 1;

            public Order1(){
                System.out.println(this);
            }

            @Override
            public String toString() {
                return "Order1 [ORDERED=" + ORDERED + "]";
            }

        }
```
​        Order2、Order3类与Order1类类似，只不过注解是@Order(2)、@Order(3)，当启动程序后Spring开始加载该三个类，日志打印如下：

​        Order1 [ORDERED=1]
​        Order2 [ORDERED=2]
​        Order3 [ORDERED=3]


# @PostConstruct注解
​        参考： https://blog.csdn.net/m0_53288098/article/details/122355201

​        类初始化调用顺序：

（1）构造方法Constructor

（2）@Autowired

（3）@PostConstruct

```java
        public class UserController {
            @Autowired
            private UserService userService;

            public UserController() {
                // 调用userService的自定义初始化方法，此时userService为null，报错
                userService.userServiceInit();
            }
        }
```

​        使用@PostConstruct注解来完成初始化，@PostConstruct注解的方法将会在UserService注入完成后被自动调用
```java
        public class UserController {
            @Autowired
            private UserService userService;

            public UserController() {
            }

            // 初始化方法
            @PostConstruct
            public void init(){
                userService.userServiceInit();
            }
        }
```

# @JsonProperty("alias")
```java
        @JsonProperty("k")
        private String key;
```
​        当序列化时会转换名字`key`为`k`
​        使用场景：数据库数据键值为`k`，但是类定义因为规范原因只能为`key`而不能使用单字母，此时需要对应，用`@JsonProperty`来转换
​        还有当输出时为了保留大小写，用别名来保持大小写


# @RequestBody
```java
        @RequestMapping("test")
        @ResponseBody
        public User test(@RequestBody User user) {
            return user;
        }

// 测试数据
        {
            "userName" : "haha",
                "age":15
        }
```

`@RequestBody`的作用：

`@RequestBody`可以<mark style="background: #FF5582A6;">接收POST请求</mark> content-type为application/json的参数，消息体中的参数为json格式，springmvc会使用MappingJackson2HttpMessageConverter将json格式的参数转换为java对象,MappingJackson2HttpMessageConverter默认支持的格式为content-type:application/json。

# @RequestParam
   `@RequestParam`可以接收`POST`请求`content-type:application/x-www-form-urlencoded`和`GET`请求url的querystring里的参数。
   <mark style="background: #FF5582A6;">不能使用封装的对象接收</mark> 参数，只能将参数单独接收，<mark style="background: #FF5582A6;">但可以将参数用一个map</mark> 接收
```java
// 不能在参数里面@RequestParam(required = false) User user 直接接收封装对象

        @RequestMapping("test")
        @ResponseBody
        public User test(@RequestParam(required = false) String userName,@RequestParam(required = false) String password) {
            User user = new User();
            user.setUserName(userName);
            user.setPassword(password);
            return user;
        }
```
​        请求参数和指定名称不一致
​                请求参数非username而是name,
` http://localhost:8080/send5?name=haha`

```
(@RequestParam(value = "name", defaultValue = "null", required = true) String username)
```


# @PathVariable
​        其用来获取请求路径（`url`）中的动态参数
```java
/**
 * @RequestMapping(value = "/person/profile/{id}/{name}/{status}") 中的 {id}/{name}/{status}
 * 与 @PathVariable int id、@PathVariable String name、@PathVariable boolean status
 * 一一对应，按名匹配。
 */
        @RequestMapping(value = "person/profile/{id}/{name}/{status}")
        @ResponseBody
        public Person porfile(@PathVariable int id, @PathVariable String name, @PathVariable boolean status) {
            return new Person(id, name, status);
        }
```



# mvn生命周期

​        https://www.runoob.com/maven/maven-build-life-cycle.html

​        报错：找不到对应符号，找不到对应的类
​        该模块引用了其他模块的类，但是其他模块没有先打包install覆盖本地新修改的类，那么引用该模块新修改的类时就会报错。




# 加载资源resource
​        第一种：
```java
        ClassPathResource classPathResource = new ClassPathResource("excleTemplate/test.xlsx");
        InputStream inputStream =classPathResource.getInputStream();
```

​        第二种：
```java
        InputStream inputStream = Thread.currentThread().getContextClassLoader().getResourceAsStream("excleTemplate/test.xlsx");
```

​        第三种：
```java
        InputStream inputStream = this.getClass().getResourceAsStream("/excleTemplate/test.xlsx");
```

# 日志
## 日志门面
​                JCL

​        slf4j 配置
​        https://logging.apache.org/log4j/2.x/manual/configuration.html

## 日志实现
​        JUL、log4j、logback、log4j2

## 注解
​        Lombok里面`@Slf4j`


# 好文收集
​                [如何优雅的写 Controller 层代码？](https://mp.weixin.qq.com/s/msKJA2aj1hVYxcwQfbLHDw )



# @min @max参数校验

​                spring-boot-start-web 2.3之后需要手动添加该包
```java
                <dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

​        参数校验时产生的错误信息传送给全局异常统一处理

```java

        @ControllerAdvice(basePackages = "com.ahang.springbootinitdemo.controller")
        @Slf4j
        public class GlobalExceptionHandler {

            // 处理ServiceException异常
            @ResponseBody
            @ExceptionHandler(value = ServiceException.class)
            public CommonResult serviceExceptionHandler (HttpServletRequest req, ServiceException ex) {
                log.debug("[serviceExceptionHandler]", ex);
                log.info("[serviceExceptionHandler]", ex);
                return CommonResult.error(ex.getCode(), ex.getMessage());
            }

            @ResponseBody
            @ExceptionHandler(value = MissingServletRequestParameterException.class)
            public CommonResult missingServletRequestParameterExceptionHandler (HttpServletRequest req, MissingServletRequestParameterException ex) {
                log.debug("[exceptionHandler]", ex);
                return CommonResult.error(ServiceExceptionEnum.MISSING_REQUEST_PARAM_ERROR.getCode(),
                        ServiceExceptionEnum.MISSING_REQUEST_PARAM_ERROR.getMessage());
            }

            @ResponseBody
            @ExceptionHandler(value = Exception.class)
            public CommonResult exceptionHandler (HttpServletRequest req, Exception e) {
                log.error("[exceptionHandler]", e);
                return CommonResult.error(ServiceExceptionEnum.SYS_ERROR.getCode(),
                        ServiceExceptionEnum.SYS_ERROR.getMessage());
            }

            @ResponseBody
            @ExceptionHandler(value = ConstraintViolationException.class)
            public CommonResult constraintViolationExceptionHandler (ConstraintViolationException e) {
                Set<ConstraintViolation<?>> constraintViolations = e.getConstraintViolations();
                StringBuffer message = new StringBuffer();
                String substring = null;
                for (ConstraintViolation<?> violation : constraintViolations) {
                    message.append(violation.getMessage());
                    message.append(";");
                }
                if (message != null) {
                    substring = message.substring(0, message.length() - 1);
                }
                log.error("[constraintViolationException]", e);
                return CommonResult.error(ServiceExceptionEnum.INVALID_REQUEST_PARAM_ERROR.getCode(),
                        substring);
            }


        }
```



# 全局统一响应

## 问题：ctroller返回类型为String时报错

​        解决：
​        方法一（优先）：重新封装一个VO对象，仅包含该参数
​                方法二
​        添加判断
```java
        if (body instanceof String) {
            return JSON.toJSONString(CommonResult.success(body));
        }
        @ControllerAdvice(basePackages = "com.ahang.springbootinitdemo.controller")
        public class GlobalResponseBodyHandler implements ResponseBodyAdvice {
            @Override
            public boolean supports(MethodParameter methodParameter, Class aClass) {
                return true;
            }

            @Override
            public Object beforeBodyWrite(Object body, MethodParameter methodParameter, MediaType mediaType,
                    Class aClass, ServerHttpRequest serverHttpRequest,
                    ServerHttpResponse serverHttpResponse) {
                if (body instanceof CommonResult) {
                    return body;
                }
                if (body instanceof String) {
                    return JSON.toJSONString(CommonResult.success(body));
                }
                return CommonResult.success(body);
            }
        }
```

## 问题：参数类型为int时默认为0

​        在`controller`中传入参数类，此时实体类中存在属性int类型时，如果传入值为空，此时会默认为0
```java
        @RestController
        public class InternationalUserController {
            @PostMapping("/id")
            public UserVo getId(@Valid UserVo userVo) {
                log.info("[getId] UserVo={}", userVo.toString());
                return userVo;
            }
        }

        public class UserVo {
            @NotNull(message = "{UserVo.id.NotNull}")
            int id; // 默认为0,这样会导致无法判断传入值为空
        }

        public class UserVo {
            @NotNull(message = "{UserVo.id.NotNull}")
            Integer id;   // 此时默认为null，可以被异常捕获
        }
```



# 国际化


# 强制规定

## controller接收参数的实体类属性必须使用包装类型
​        防止出现使用基本类型int时出现，传入为null或没传时，却给了默认值0，导致异常没有被检测




# RestTemplate使用

```java
        String url="http://www.test.com/api/getUser";
        User user= new User();
        user.setName("zhangsan");

        String s = JSON.toJSONString(user);
// 要点1: 这三句设置header非常重要
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON); // 规定了请求体body的类型
        HttpEntity<String> request = new HttpEntity<String>(s, headers);

        try {
            // 要点2：这里要发送包含header的request，不要直接发对象的json报文
            restTemplate.postForEntity(url, request, User.class);
        }catch (Exception e){
            logger.error("异常了",e);
        }

```







# Stream流式编程

## 创建

​                **（1）使用指定值创建Stream实例**

```java
// of为Stream的静态方法 
        Stream<String> strStream = Stream.of("hello", "java8", "stream"); 
// 或者使用基本类型流
        IntStream intStream = IntStream.of(1, 2, 3);
```

**（2）使用集合创建Stream实例（常用方式）**

```java
// 使用guava库，初始化一个不可变的list对象 
        ImmutableList<Integer> integers = ImmutableList.of(1, 2, 3); 
// List接口继承Collection接口，java8在Collection接口中添加了stream方法 
        Stream<Integer> stream = integers.stream(); 
```

**（3）使用数组创建Stream实例**

```java
// 初始化一个数组 
        Integer[] array = {1, 2, 3}; 
// 使用Arrays的静态方法stream 
        Stream<Integer> stream = Arrays.stream(array); 
```

**（4）使用生成器创建Stream实例**

```java
// 随机生成100个整数 
        Random random = new Random(); 
// 加上limit否则就是无限流了 
        Stream<Integer> stream = Stream.generate(random::nextInt).limit(100); 
```

**（5）使用迭代器创建Stream实例**

```java
// 生成100个奇数，加上limit否则就是无限流了 
        Stream<Integer> stream = Stream.iterate(1, n -> n + 2).limit(100); 
        stream.forEach(System.out::println); 
```

**（6）使用IO接口创建Stream实例**

```java
// 获取指定路径下文件信息，list方法返回Stream类型
        Stream<Path> pathStream = Files.list(Paths.get("/")); 
```


[参考](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html)


​        ![](https://img.liboliu.com/2021-06-14/372acd67-5b57-490f-af46-f9ab859d2846.png)
​        Stream流操作的有状态 vs 无状态
​        比如map或者filter会从输入流中获取每一个元素，并且在输出流中得到一个结果，这些操作没有内部状态，称为无状态操作。
​        但是像reduce、sum、max这些操作都需要内部状态来累计计算结果，所以称为有状态操作。

​        这里需要单独解释一下：
​        有一些操作sort、distinct、limit、skip看上去和filter、map差不多，他们接收一个流，再生成一个流，但是区别在于排序和去重复项需要知道先前的历史。比如排序就需要将所有元素放入缓存区后才能给输出流加入一个项目，这个操作对缓存的要求是无上限的，流有多大就需要多大的缓存才能进行运算。这些操作也是有状态操作。

​        短路与非短路操作
​        短路：匹配到前部分满足了条件就会停止
​        非短路：匹配全部

## 中间状态

`sort`
        Stream流中间状态不会立即执行语句，是懒执行
        直到使用了最终状态时才会执行中间状态的语句。
```java
        ArrayList<Integer> numbers = new ArrayList<>(Arrays.asList(2,3,4,5));
        Stream<Integer> stream = numbers.stream().sorted( (a,b)->b.compareTo(a) );

// 在已经定义好的流使用的列表中，仍添加数字后，结果仍然会排好序
        numbers.add(3);
        numbers.add(1);
        stream.forEach(System.out::print); // 5 4 3 3 2 1
distinct
        ArrayList<Integer> numbers = new ArrayList<>(Arrays.asList(2,3,3,5));
        numbers.stream().distinct().forEach(System.out::print); // 235
```

`filter`
        根据指定条件过滤：如字符串长度，数字大小
```java
        ArrayList<String> numbers = new ArrayList<>(Arrays.asList("aa", "b", ""));
        numbers.stream().filter(s -> s.length() > 0).forEach(System.out::print ); // aab 
```

`map`
        对每个元素都映射操作
```java
        ArrayList<Integer> numbers = new ArrayList<>(Arrays.asList(2,3,5));
        numbers.stream().map(a -> a*a).forEach(System.out::print); // 4925
```

`limit`取前几个
`skip`跳过前几个
```java
        ArrayList<Integer> numbers = new ArrayList<>(Arrays.asList(2,3,3,5));

        numbers.stream().limit(3).forEach(System.out::print);  // 233
        numbers.stream().skip(1).forEach(System.out::print);  // 335
```

​        流操作具体流程示意图
```java
        List<String> strings = Arrays.asList("Hollis", "HollisChuang", "hollis", "Hello", "HelloWorld", "Hollis");
        Stream s = strings.stream()
                .filter(string -> string.length()<= 6)
                .map(String::length)
                .sorted()
                .limit(3)
                .distinct();
```
​        ![](https://www.hollischuang.com/wp-content/uploads/2019/03/15521242025506.jpg)


## 终结状态

​        最终操作会消耗流，产生一个最终结果。也就是说，<mark style="background: #FF5582A6;">在最终操作之后，不能再次使用该流</mark> ，也不能再使用任何中间操作，否则将抛出异常：

```java
        java.lang.IllegalStateException: stream has already been operated upon or closed
```

**forEach**

​        Stream 提供了方法 `forEach` 来迭代流中的每个数据。以下代码片段使用 `forEach` 输出了10个随机数：

```java
        Random random = new Random();
        random.ints().limit(10).forEach(System.out::println);
```

**count**

​        count用来统计流中的元素个数。

```java
        List<String> strings = Arrays.asList("Hollis", "HollisChuang", "hollis", "Hollis666", "Hello", "HelloWorld", "Hollis");
        System.out.println(strings.stream().count());
//7
```

**collect**

​        collect就是一个归约操作，可以接受各种做法作为参数，将流中的元素累积成一个汇总结果（可以转换成set,map,list等）：

```java
        List<String> strings = Arrays.asList("Hollis", "HollisChuang", "hollis","Hollis666", "Hello", "HelloWorld", "Hollis");
        strings  = strings.stream().filter(string -> string.startsWith("Hollis")).collect(Collectors.toList());
        System.out.println(strings);
//Hollis, HollisChuang, Hollis666, Hollis
        ArrayList<String> str = new ArrayList<String>(Arrays.asList("haha", "hello","hhhh","ahang"));
        Stream<String> strStream= str.stream().filter(s -> s.length() == 4);
        List<String> lists = strStream.collect(Collectors.toList());   // collect(Collectors.toList()) 转化Stream为List集合
        Set<String> sets = strStream.collect(Collectors.toSet());   // collect(Collectors.toSet()) 转化Stream为Set集合

        String[] names = {"haha,20", "ahang,30","kaka,23"};
        Stream<String> arrayStream = Stream.of(names);
        // collect( Collectors.toMap(Function keyMapper,Function valueMapper))转化Stream为map集合
        Map<String, Integer> map = arrayStream.collect( Collectors.toMap(s->s.split(",")[0], s-> Integer.parseInt(s.split(",")[1]))  );
        Set<String> keySet = map.keySet();
        for(String key: keySet){
            Integer value = map.get(key);
            System.out.println(key + ":" + value);
        }
```

​        接下来，我们还是使用一张图，来演示下，前文的例子中，当一个Stream先后通过filter、map、sort、limit以及distinct处理后，在分别使用不同的最终操作可以得到怎样的结果：

​        下图，展示了文中介绍的所有操作的位置、输入、输出以及使用一个案例展示了其结果。
​        ![](https://www.hollischuang.com/wp-content/uploads/2019/03/15521245463720.jpg)

# Optional
​        原文链接：https://blog.csdn.net/NoxUni/article/details/107311593

## 概述
​        可以为null的容器对象，可以保存类型T的值
​                可以解决空指针异常
## 创建Optional实例
​                创建空的Optional类
`Optional<User> emptyOpt = Optional.empty();`

​        Of()方法：返回一个指定非 null 值的 Optional。若传入 null，会抛出 NullPointerException
`Optional<User> opt = Optional.of(user)`

​        OfNullable()方法：如果入参非空，返回 Optional 描述的指定值，否则返回空的 Optional。该方法可以解决空指针异常问题
`Optional<User> opt = Optional.ofNullable(user);`

## 验证是否有值
​        有如下两种验证方法：

```
boolean isPresent()`：有值为 true，无值为 false
`void ifPresent(Consumer<? super T> consumer)`：接受一个Consumer参数，如果值存在则使用该值调用 consumer，否则不做任何事
// 如果入参不为空，打印age
`opt.ifPresent(u -> System.out.println(u.getAge()));
```


## 从Optional中返回值
`T get()`：若 opt 为空，抛出 NoSuchElementException
```java
        User user = new User("Harry", 11);
        Optional<User> opt = Optional.ofNullable(user);
        opt.get().getName();
```

`T orElse(T other)`：如果存在值则返回值，否则返回 other
```java
        User user1 = null;
        User user2 = new User("Harry", 11);
        User resultA = Optional.ofNullable(user1).orElse(user2); // 返回user2

        User user3 = new User("Ron", 12);
        User resultB = Optional.ofNullable(user2).orElse(user3); // 返回user2
        T orElseGet(Supplier<? extends T> other)：如果存在值则返回值，否则执行入参的 Supplier参数，返回其执行结果
        User result = Optional.orNullable(user).orElseGet(() -> user2);
```


`<X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier)`：存在值则返回值，否则抛出由 Supplier 继承的异常
        User result = Optional.ofNullable(user).orElseThrow(() -> new IllegalArgumentException());

## 转换值
`<U>Optional<U> map(Function<? super T,? extends U> mapper)`：如果有值，则对其执行调用映射函数得到返回值。如果返回值不为 null，则创建包含映射返回值的Optional作为map方法返回值，否则返回空Optional。
```java
        User user1 = new User("Harry", 11);
        User user2 = null;
        String name1 = Optional.ofNullable(user1).map(u -> u.getName()).orElse("John Doe"); // 返回Harry
        String name2 = Optional.ofNullable(user2).map(u -> u.getName()).orElse("John Doe"); // 返回John Doe
```

`<U> Optional<U> flatMap(Function<? super T,Optional<U>> mapper)`：如果值存在，返回基于Optional包含的映射方法的值，否则返回一个空的Optional。
        与 map() 的区别在于，map方法返回的值应为 Optional 类型
        String name = Optional.ofNullable(user).flatMap(u -> u.getOptName()).orElse("default");

​        User类中的 getOptName() 方法：
```java
        public Optional<String> getOptName(){
            return Optional.ofNullable(name);
        }
```

## 过滤值
```
Optional<T> filter(Predicate<? super <T> predicate)`：如果值存在，并且这个值匹配给定的 predicate，测试结果为 true 则返回一个 Optional 用以描述这个值，否则返回一个空的Optional。
`Optional<User> result = Optional.ofNullable(user).filter(u -> u.getName() != null && u.getAge() > 0 && u.getAge() < 100);
```

​        参考：
​        https://www.cnblogs.com/zhangboyu/p/7580262.html
​        https://www.runoob.com/java/java8-optional-class.html

# 查看类加载器

​        在`DEBUG`时通过 执行器 查看
`ClassLoader.getSystemClassLoader()`

```
Thread.currentThread().getContextClassLoader()`
`ClassLoader.getSystemClassLoader()
```

# Assert断言

## Java的assert
​        1、`assert <boolean表达式>`
​        如果<boolean表达式>为true，则程序继续执行。
​        如果为false，则程序抛出AssertionError，并终止执行。
​        2、`assert <boolean表达式> : <错误信息表达式>`
​        如果<boolean表达式>为true，则程序继续执行。
​        如果为false，则程序抛出java.lang.AssertionError，并输入<错误信息表达式>。
​        `assert "a".equals("ab") : "断言失败，此表达式的信息将会在抛出异常的时候输出！";`


## Junit的Assert
​        以下方法在`import org.junit.Test;`直接使用
​        或者通过`import org.junit.Assert;`，然后`Assert.assertEquals(Object expected, Object actuals);`

`assertEquals` 和 `assertTrue` 区别
        相同之处：都能判断两个值是否相等
        assertTrue 如果为true，则运行success，反之Failure
        assertEquals 如果预期值与真实值相等，则运行success，反之Failure

​        不同之处：
<mark style="background: #FF5582A6;">assertEquals 运行Failure会有错误提示</mark> ，提示预期值是xxx，而实际值是xxx。容易调式
​        assertTrue 没有错误提示

​        判断两数组是否相同：`assertArrayEquals(String message, int[] expecteds, int[] actuals)`




# 算法
​        默认优先级队列为小顶堆：
​        所以`min.peek(), min.poll()`都是操作堆顶最小值(所以弹出顺序是从小到大)
`PriorityQueue<Integer> min = new PriorityQueue<>();`
`PriorityQueue<Integer> min = new PriorityQueue<Integer>(4, (o1, o2) -> o1 - o2);`


​        修改为：固定空间大小，大顶堆
`PriorityQueue<Integer> queue = new PriorityQueue<Integer>(k, ((o1, o2) -> o2-o1) );`



# 多态

`public class Dog extends Animal`
`public class Cat extends Animal`
`method方法`使用`Animal`父类去接收子类，返回值使用`Animal`父类去返回子类
```java
        public class ExtendMain {
            public static void main(String[] args) {
                Dog dog = new Dog("10");
                dog.setEat("1");
                Cat cat = new Cat("20");
                cat.setEat("2");
//        Animal animal = method(dog);  
                Animal animal1 = method(cat);
                Cat animal2 = (Cat) method(cat);
            }

            public static Animal method (Animal animal) {
                if (animal.getClass().isInstance(Dog.class)) {
                    Dog dog = (Dog) animal;
                    return dog;
                } else {
                    Cat cat = (Cat) animal;
                    return cat;
                }
            }
        }
```
# 泛型
`String.class` 实际上是一个：`Class<String>` 类的对象


# 双亲委派

```java
        public Class<?> loadClass(String name) throws ClassNotFoundException {
            return loadClass(name, false);
        }
        //              -----??-----
        protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
        {
            // 首先，检查是否已经被类加载器加载过
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                try {
                    // 存在父加载器，递归的交由父加载器
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        // 直到最上面的Bootstrap类加载器
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            return c;
        }
```

​        ![](https://img-blog.csdnimg.cn/20201217213314510.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NvZGV5YW5iYW8=,size_16,color_FFFFFF,t_70)

​        机制好处
                - 防止核心库类不会被自己写的篡改
                - 防止重复加载相同类

​        ![](https://img-blog.csdnimg.cn/2020121722082798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NvZGV5YW5iYW8=,size_16,color_FFFFFF,t_70)


# Enum枚举
​        常用法：先定义一对值，表示状态码和代表的信息

```java
        @Getter
        public enum HttpStatusEnums {
            SUCCESS("200", "success"),
            BAD_REQUEST("400", "BAD"),
            UNAUTHORIZED("401", "Unauthorized"),
            SERVER_NOT_FOUND("404", "接口不存在");

            private final String code;
            private final String message;

            HttpStatusEnums(String value, String message) {
                this.code = value;
                this.message = message;
            }
        }
```
​        然后使用时，通过`getCode`和`getMessage`来调用
```java
        public static void main(String[] args) {
            String code = "200";
            if (HttpStatusEnums.SUCCESS.getCode() == code) {
                System.out.println(HttpStatusEnums.SUCCESS.getMessage());
            } else {
                System.out.println(HttpStatusEnums.BAD_REQUEST.getMessage());
            }
        }
```


​        如果是常用常量使用接口
```java
        public interface Constants {
            String X_XSRF_TOKEN = "X-XSRF-TOKEN";
            Integer MAX_PAGE = 10;
        }
```
​        使用
```java
        public static void main(String[] args) {
            int page = Constants.MAX_PAGE;
        }
```





​        cglib代理
​                注解
​        多线程查看
​                jackson使用




# 基础
  `strURL = strURL.replaceAll("%3A", ":").replaceAll("%2F", "/").replaceAll("%3F", "?").replaceAll("%3D", "=").replaceAll("%26", "&"); `

  转换的原理。
   `: -> 3A -> 16*3+10 -> 58 -> chr(58) = ":"` 
   `/ -> 2F -> 16*2+15 -> 47 -> chr(47) = "/"`
    ----------------------------------------------------- 
  >   16*高位+低位 3A（16进制）→58（10进制）→字符（58）→显示“：” 编码，不是 C++，这个是将UTF8转换成ANSI编码。



# mybatis

​        今天在使用MyBatis的时候，看到了一个
`<include refid="Base_Column_List" />`
​        的语句。
​        在数据库的使用中，查询的时候时候不要使用`*`号，`*`是查询所有，这样如果改动表，或者对查询的效率都有非常大的影响，

​        而查询的语句更推荐些出相对应的字段比如
​                反例

```
SELECT * FROM T_USER
```

​        推荐
`SELECT ID,USER,PSW FROM T_USER`

​        在MyBatis中有很多的查询语句，如果每个都列出字段，显得十分麻烦，这样就可以有include的用法。

​        列出例子：
​        写出固定的前面字段
​        ![](https://img-blog.csdn.net/20171025100134645?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTQxNzExNzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

​        ![](https://img-blog.csdn.net/20171025100220148?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTQxNzExNzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

​        原文链接：https://blog.csdn.net/a4171175/article/details/78337726


​        ![](https://img.jbzj.com/file_images/article/202102/20210219145910.jpg)

​        https://www.jb51.net/article/205952.htm
​        https://www.jb51.net/article/214460.htm




# 多线程

## CountDownLatch

 CountDownLatch非常适合于对任务进行拆分，使其并行执行，比如某个任务执行2s，其对数据的请求可以分为五个部分，那么就可以将这个任务拆分为5个子任务，分别交由五个线程执行，执行完成之后再由主线程进行汇总，此时，总的执行时间将决定于执行最慢的任务，平均来看，还是大大减少了总的执行时间。

`CountDownLatch`允许一个或者多个线程去等待其他线程完成操作。

​        https://blog.csdn.net/hbtj_1216/article/details/109655995



# Json

​        @JsonProperty
​        在序列号和反序列化时使用。
​        作用于属性上，作用是把该属性的名称序列化成另一个自己想要的名称。
```java
        @JsonProperty("name")
        private String trueName; // 假如 trueName 最后为"小明"
// 转化为 json 后: {"name":"小明"} 
```
————————————————
        @JsonIgnoreProperties
        选择性忽略类中的属性，通常作用于类上。
```java
        @JsonIngoreProperties(value={"name","sex"})
        public class Person{
            private String name;
            private String pwd;
            private String sex;
            private Integer age;  
```
​        ————————————————
​            @JsonFormat
​            格式转换
```java
            @JsonFormat(timezone="GTM+8",pattern="yyyy-MM-dd HH:mm:ss")
            private Date createDate;
```


        # 算法

#### [154. 寻找旋转排序数组中的最小值 II](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array-ii/)








# IDEA
​        问题： `ctrl + shift + f` 搜索项目文件失效
​        解决：与Windows的繁简切换快捷键冲突

​                调试工具debug
​        ![](https://img-blog.csdnimg.cn/20200727114157536.png)

**step over**
        ![](https://img-blog.csdnimg.cn/20200727114752368.png)
        (F8)：步过，一行一行地往下走，如果这一行上有方法不会进入方法。
        在单步执行时，在函数内遇到子函数时<mark style="background: #FF5582A6;">不会进入子函数内单步执行</mark> ，而是将子函数整个执行完在停止，也就是把子函数整个作为一步；

**step into**
        ![](https://img-blog.csdnimg.cn/20200727114915618.png)
        步入，如果当前行有方法，可以进入方法内部，一般用于进入自定义方法内，不会进入官方类库的方法。单步执行，<mark style="background: #FF5582A6;">遇到子函数就进入并且继续单步执行</mark> ；

**step out**
        ![](https://img-blog.csdnimg.cn/20200727114955774.png)
        (Shift + F8)：
        单步执行到子函数内时，用step out就可以<mark style="background: #FF5582A6;">执行完子函数余下部分，并返回到上一层</mark> 函数。

# 问题
## 问题：语法检查 `Cannot resolve symbol`
​        解决：
        - “File” -> “Invalidate Caches / Restart”
        - File - Project Structure - Project SDK
                - File - Settings - 搜索[maven](https://so.csdn.net/so/search?q=maven&spm=1001.2101.3001.7020)，Maven home directory，设置为自己安装的maven路径


# 插件推荐

`Alibaba Java Coding Guidelines`检查java代码风格

`leetcode Editor`刷题

```
maven helper
mybatisCodeHelp
```

`openAPI(swagger)` swagger文档查看

`Translation` 翻译

`GsonFormat-plus` 根据返回json结果，转化为对应实体类，`alt + s`使用




# 热部署



​        https://www.iocoder.cn/Spring-Boot/hot-swap/
