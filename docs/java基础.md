[TOC]



## 1. 常量
用`final`修饰，一旦定义后面就不能修改了，如`final double MAX_D = 100.101`

1.  final 修饰一个引用
    -   如果引用是基本数据类型，则该引用为常量，该值无法修改。
    -   如果引用为引用数据类型，比如对象、数组，则<mark style="background: #BBFABBA6;">该对象、数组本身可以修改</mark> ，但指向该对象或数组的<mark style="background: #FF5582A6;">地址的引用不能修改</mark> 。
    -   如果引用是类的成员变量，则必须当场赋值，否则编译会报错。
2.  final 修饰类中的方法
    当使用 final 修饰方法时，这个方法将成为最终方法，<mark style="background: #BBFABBA6;">无法被子类重写</mark> 。但是，<mark style="background: #FF5582A6;">该方法仍然可以被继承。</mark> 
3.  final 修饰类
    当用 final 修改类时，该类成为最终类，<mark style="background: #FF5582A6;">无法被继承</mark> 。简称为“断子绝孙类”。比如常用的 String 类就是最终类。

定义一个类常量：在一个类中多个方法都能调用的常量
用`static final` 修饰

```java
public class Lei{
    public static final int AAA = 10;
    
    public static void main(){ int a = AAA }
    public static void method(){ int b = AAA }
}
```



Java没有无符号（unsigned）形式long、int、short、byte类型

默认浮点数8字节为double，后缀D、d

4字节float后缀F、f

​	

数学函数`Math`类下函数`Math.PI, Math.E, Math.sqrt()`

随机数：`Math.random()`



低精度向高精度自动转型

强制转型：`(int)a`



大数值

`BigInteger bi = BigInteger.valueOf(100);`

`BigInteger`, `BigDecimal`

运算通过`add()`,`multiply()`,`toString`

## 2. 运算符

### 2.1. 异或（XOR，^）：不同为1，相同为0。

特殊用法：使特定数位翻转（与1异或）、保留原值（与0异或）、交换两个变量的值。

 交换两个变量的值：
- 借助第三个变量来实现。`C=A;A=B;B=C`;
- 利用加减法实现两个变量的交换。`A=A+B;B=A-B;A=A-B`;
- 用位异或运算来实现，效率最高。`A=A^B;B=A^B;A=A^B`;(==一个数异或本身等于0==)


### 2.2. 负数以其正值的补码形式表示

原码：一个整数按照绝对值大小转换成的二进制数称为原码。

`[+1]原= 0000 0001`

`[-1]原= 1000 0001`

反码：将二进制按位取反，所得的新二进制数称为原二进制数的反码。

`[+1] = [0000 0001]原= [0000 0001]反`

`[-1] = [1000 0001]原= [1111 1110]反`

补码的表示方法是，正数的补码就是其本身，负数的补码是在其原码的基础上，符号位不变，其余各位取反，最后+1（也就是在其反码的基础上+1）

`[+1] = [0000 0001]原= [0000 0001]反= [0000 0001]补`

`[-1] = [1000 0001]原= [1111 1110]反= [1111 1111]补`


### 2.3. 左移运算：`value<<num`

1）数值value向左移动num位，左边二进制位丢弃，右边补0。（注意byte和short类型移位运算时会变成int型，结果要强制转换）

2）若1被移位到最左侧，则变成负数。

3）左移时舍弃位不包含1，则左移一次，相当于乘2。

### 2.4. 右移运算：`value>>num`

1）数值value向右移动num位，正数左补0，负数左补1，右边舍弃。（即保留符号位）

2）右移一次，相当于除以2，并舍弃余数。

3）无符号右移>>>：左边位用0补充，右边丢弃

**计算机内的数值表示为补码二进制**

-5换算成二进制： `1111 1111 1111 1111 1111 1111 1111 1011`

-5右移3位后结果为-1，-1的二进制为：  `1111 1111 1111 1111 1111 1111 1111 1111`   // (用1进行补位)

-5无符号右移3位后的结果 536870911 换算成二进制：  `0001 1111 1111 1111 1111 1111 1111 1111`   // (用0进行补位)


### 2.5 运算符优先级

| 优先级 | 运算符                                             | 结合性   |
| ------ | -------------------------------------------------- | -------- |
| 1      | ()、[]、{}                                         | 从左向右 |
| 2      | !、+(一元运算符)、-、~、++、--                     | 从右向左 |
| 3      | *、/、%                                            | 从左向右 |
| 4      | +、-                                               | 从左向右 |
| 5      | <<、>>、>>>                                        | 从左向右 |
| 6      | <、<=、>、>=、instanceof                           | 从左向右 |
| 7      | ==、!=                                             | 从左向右 |
| 8      | &                                                  | 从左向右 |
| 9      | ^                                                  | 从左向右 |
| 10     | \|                                                 | 从左向右 |
| 11     | &&  短路与，第一个为false则不再计算                | 从左向右 |
| 12     | \|\|  短路或，第一个为true则不再计算               | 从左向右 |
| 13     | ?:                                                 | 从右向左 |
| 14     | =、+=、-=、*=、/=、&=、\|=、^=、~=、<<=、>>=、>>>= | 从右向左 |

```java
int n=7; n<<=3; 

n=n & n + 1 | n + 2 ^ n + 3; 
// 先+ - 运算：n = 56 & 57 | 58 ^ 59  
// 然后 按照 & ^ | 顺序运算
n>>=2;
```

## 3. 字符串

### 3.1 子串
```java
String str = "hello";
String substr = str.substring(0,3);  // 从0到2的子串: hel
```

### 3.2 拼接
```java
        String a = "Hello";
        String b = "world";
        String c = a + b;   // 变量通过 +号 拼接
        String d = "hello " + b;  // 字符串 + 变量
        System.out.println("Hei "+ b);   // 在输出时通过 +号 拼接
        String all = String.join("/", "M","L","XL","XXL"); // 第一个为分隔符：M/L/XL/XXL
```

### 3.3 拆分split
`split() `方法根据匹配给定的正则表达式来拆分字符串。

注意：` . 、 $、 | 和 *` 等转义字符，必须得加 `\\`。

注意：多个分隔符，可以用 `|` 作为连字符。

```java
        a = "ab,bc,cd-d-e";
        b = "192.168.10.1";
        
        String[] al = a.split(",|-");
        for(String tmp1: al) System.out.println(tmp1);  
        
        for(String tmp2: b.split("\\.")) System.out.println(tmp2);
```

- `toLowerCase()` 转化为小写
- `toUpperCase()` 转化为大写
- `trim()` 删除首尾空格



**不要用`==`运算符检查两个字符串是否相同**，只是检查地址

用`str1.equals(str2)`



### 3.4 通过StringBuilder构建

#### 3.4.1 StringBuilder --> String
避免拼接很多小的字符串都要声明一个String来存储，浪费时空。
通过StringBuilder构建更快相当于放入缓存，当使用toString()方法时才声明一个String。
```java
        StringBuilder builder = new StringBuilder();  // 构建StringBuilder
        builder.append(a);       // 对其操作
        builder.append("abcd");
        
        String str1 = builder.toString();  // 生成一个String类
        
        System.out.println(str1);
        builder.insert(1,"hhh");
        String str2 = builder.toString();
        System.out.println(str2);
        builder.delete(2,5);
        String str3 = builder.toString();
        System.out.println(str3);
```

#### 3.4.2 字符串逆转
- `StringBuilder`有一个`reverse()`方法可以逆转字符串，但是`String`类没有该方法

#### 3.4.3 String --> StringBuilder
通过构造方法：`StringBuilder(String str)`
```java
        String s1 = "haha";
        StringBuilder s2 = new StringBuilder(s1);  // 构造时转变
        s2.reverse();    // 逆转字符串
        System.out.println(s2);
```

### 3.5 字符串比较`==`和`equals`

- `==`比较的是两个字符串的地址是否相同
- `equals`比较的是两个字符串的对象是否相同

```java
// new的数值在堆中，开辟了两个新的地址来存储，所以==比较不同
    String a = new String("aaaa");
    String b = new String("aaaa");
    System.out.println("a==b: " + (a==b));  // false
    System.out.println("a.equals(b): "+ a.equals(b)); // true
    
// 除了堆栈外有个常量池，字符串放入常量池，第一个写入，第二个相同的直接将地址引用相同的，所以==相同
    String a = "aaaaa";
    String b = "aaaaa";
    String c = a;      // 此时==得到的为true，指向同一个地址 
    System.out.println("a==b: "+ (a==b));  // true
    System.out.println("a.equals(b): "+ a.equals(b));  // true
```

### 3.6 字符串空判断

`StringUtils.isEmpty()`同时判断了`String`是否为`null`和`""`，其中`null != ""`

所以一般判断传入的字符串是否有效时使用`StringUtils.isEmpty( String )`而不是`String != null`





## 4. ArrayList集合

- 什么是集合

提供一种存储空间可变的存储模型，存储的数据容量可以发生改变
- ArrayList 集合的特点

底层是数组实现的，长度可以变化
- 泛型的使用

用于约束集合中存储元素的数据类型

### 4.1 常用方法
| 方法名                               | 说明                                   |
| ------------------------------------ | -------------------------------------- |
| public boolean remove(Object o)      | 删除指定的元素，返回删除是否成功       |
| public E remove(int index)           | 删除指定索引处的元素，返回被删除的元素 |
| public E set(int index,E element)    | 修改指定索引处的元素，返回被修改的元素 |
| public E get(int index)              | 返回指定索引处的元素                   |
| public int size()                    | 返回集合中的元素的个数                 |
| public boolean add(E e)              | 将指定的元素追加到此集合的末尾         |
| public void add(int index,E element) | 在此集合中的指定位置插入指定的元素     |
```java
        ArrayList<String> as1 = new ArrayList<>();  // 声明集合类型为String类型
        as1.add("hahah");   // 添加默认放最后
        as1.add(s1);
        String s3 = as1.get(1);  // 获取集合元素
        System.out.println(s3);
        if ( as1.remove("hahah")) as1.add(0,"gai");  // remove删除元素，add(index, element) 添加至指定位置
        as1.set(1,"ahaha");  // 替换原有元素

        for (int i = 0; i < as1.size(); i++) {  // size()获取大小,遍历集合
            System.out.println(as1.get(i));
        }
```

### 4.2 初始化的几种方式
- 法一：asList
```java
ArrayList<T> obj = new ArrayList<T>(Arrays.asList(Object o1, Object o2, Object o3, ....so on));

Integer[] nums = new Integer[]{3,2,5,1};
ArrayList<Integer> al = new ArrayList<Integer>(Arrays.asList(nums));

String data = "1,2,3,4";
String[] strings = data.split(",");
ArrayList<String> list = new ArrayList<String>(Arrays.asList(strings));
```
- 法二：匿名内部类
```java
ArrayList<T> obj = new ArrayList<T>() {{
    add(Object o1);
    add(Object o2);
    ...
    ...
}};
```
- 法三：传统初始化
```java
ArrayList<T> obj = new ArrayList<T>();
obj.add("o1");
obj.add("o2");
...
...

// 或者
ArrayList<T> obj = new ArrayList<T>();
List list = Arrays.asList("o1","o2",...);
obj.addAll(list);
```
- 法四：指定初始化的元素个数和值
```java
ArrayList<T> obj = new ArrayList<T>(Collections.nCopies(count,element));
//把element复制count次填入ArrayList中
```

### 4.3 List初始化

注意：无法将`int[]`数组转化为`List<Integer>`，只支持引用类型，不支持int基本类型

`Arrays.asList( new int[]{1,2,3} )`错误

```java
int[] intArray = new int[]{1, 2, 3};
Integer[] integerArray = new Integer[]{1, 2, 3};
 
List<int[]> intArrayList = Arrays.asList(intArray);  // 直接将数组转化list
List<Integer> integerList = Arrays.asList(integerArray); 
List<Integer> integerList2 = Arrays.asList(1, 2, 3);  // 直接数值初始化成list
List<String> stringList = Arrays.asList("a", "b", "c");  // 直接字符初始化为list
```



```java
        System.out.println("------ArrayList 创建-------");
        ArrayList<Integer> al = new ArrayList<>(Arrays.asList(1, 2, 3, 4));
        System.out.println(al);

        System.out.println("-----List创建------");
        int[] ints =new int[]{1, 2, 3, 5};
        int[] ints1 = {2,3,4,5};
        System.out.println(ints);  // [I@2c9f9fb0
        System.out.println(Arrays.toString(ints1)); //[2, 3, 4, 5]
        System.out.println(Arrays.asList(ints));  // [[I@2c9f9fb0]

        List<int[]> list = Arrays.asList(ints);
        List<Integer> list1 = Arrays.asList(new Integer[]{5,3,2,1});
        System.out.println(list);  // [[I@2c9f9fb0]
        System.out.println(list1);  // [5, 3, 2, 1]

        List<Integer> list3 = Arrays.asList(1, 2, 3);
        List<String> list4 = Arrays.asList("a","bb","c");
        System.out.println("list3" + list3);  // [1, 2, 3]
        System.out.println("list4" +list4);  // [a, bb, c]
```



### 4.4 Arrays排序

通过方法`sort(T[] a, Comparator<? super T> c)`，设置比较器
```java
    String[] ss = {"a","ddd","bbbb","cc"};
    System.out.println(Arrays.toString(ss)); // [a, ddd, bbbb, cc]

    Arrays.sort(ss); // 默认排序按照ASCII码排序
    System.out.println(Arrays.toString(ss)); // [a, bbbb, cc, ddd]

    Arrays.sort(ss, (a,b) -> a.length()-b.length()); // 通过lambda实现Compareto接口以实现按照字符串长度排序
    System.out.println(Arrays.toString(ss)); // [a, cc, ddd, bbbb]
```
```java
    ArrayList<String> arr = new ArrayList<String>(Arrays.asList("bbbb","a","ddd","cc"));
    System.out.println(arr);  // [bbbb, a, ddd, cc]
    Collections.sort(arr, (a,b)->a.length() - b.length());
    System.out.println(arr);  // [a, cc, ddd, bbbb]
```

## 5. 集合

集合类型分为`Collection`和`Map`两大类
![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/5-1912051036333V.png)
![Map](http://c.biancheng.net/uploads/allimg/191205/5-191205103G5960.png)

Collection集合的常用方法
```java
//创建Collection集合的对象
Collection<String> c = new ArrayList<String>();
```
| 方法名                     | 说明                               |
| -------------------------- | ---------------------------------- |
| boolean add(E e)           | 添加元素                           |
| boolean remove(Object o)   | 从集合中移除指定的元素             |
| void clear()               | 清空集合中的元素                   |
| boolean contains(Object o) | 判断集合中是否存在指定的元素       |
| boolean isEmpty()          | 判断集合是否为空                   |
| int size()                 | 集合的长度，也就是集合中元素的个数 |



迭代查询：**注意不能直接对原数据删除**

```java
		Collection<String> col1 = new HashSet<String>();
        col1.add("haha");
        col1.add("gaga");
        col1.add("wawa");

        Iterator<String> it = col1.iterator();
        while (it.hasNext()){
            String e = it.next();
            if(e.equals("wawa")){
                it.remove(); // 该方法删除是上一次next()方法返回的元素，没有问题
               // col1.remove(e);  用该方法直接对col1中数据删除会报错
            }
        }
```



使用条件删除`removeIf`,通过`lambda`表达式

```java
        Collection<String> cs = new ArrayList<String>();
        cs.add("aaa");
        cs.add("bbbbbb");
        cs.add("cc");
        cs.add("eeeeeeeeeee");
        cs.removeIf(e->e.length() > 6);  // 删除元素长度大于6的
        System.out.println(cs);
```







### 5.1 列表List

- List集合概述和特点
- List集合概述
    - ==有序==集合(也称为序列)，用户可以精确控制列表中每个元素的插入位置。用户可以通过整数索引访问元素，并搜索列表中的元素
    - 与Set集合不同，列表通常==允许重复==的元素
- List集合特点
    - 有索引
    - 可以存储重复元素
    - 元素存取有序

| 方法名                        | 描述                                   |
| ----------------------------- | -------------------------------------- |
| void add(int index,E element) | 在此集合中的指定位置插入指定的元素     |
| E remove(int index)           | 删除指定索引处的元素，返回被删除的元素 |
| E set(int index,E element)    | 修改指定索引处的元素，返回被修改的元素 |
| E get(int index)              | 返回指定索引处的元素                   |


```java
// 声明
List<Student> list = new ArrayList<Student>();

//迭代器：集合特有的遍历方式
Iterator<Student> it = list.iterator();
while (it.hasNext()) {
    Student s = it.next();
    System.out.println(s.getName()+","+s.getAge());
}

//普通for：带有索引的遍历方式
for(int i=0; i<list.size(); i++) {
    Student s = list.get(i);
    System.out.println(s.getName()+","+s.getAge());
}

//增强for：最方便的遍历方式, 但是只能查看，无法修改
for(Student s : list) {
    System.out.println(s.getName()+","+s.getAge());
}
```

#### 5.1.1 List实现类ArrayList

- ArrayList集合
    - 底层是数组结构实现，查询快、增删慢
- LinkedList集合
    - 底层是链表结构实现，查询慢、增删快

`ArrayList`类包含Collection类的所有方法，还有常用的方法
表 1 ArrayList类的常用方法

| 方法名称                                      | 说明                                                                                                                                                                      |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| E get(int index)                              | 获取此集合中指定索引位置的元素，E 为集合中元素的数据类型                                                                                                                  |
| int index(Object o)                           | 返回此集合中第一次出现指定元素的索引，如果此集合不包含该元素，则返回 -1                                                                                                   |
| int lastIndexOf(Object o)                     | 返回此集合中最后一次出现指定元素的索引，如果此集合不包含该元素，则返回 -1                                                                                                 |
| E set(int index, Eelement)                    | 将此集合中指定索引位置的元素修改为 element 参数指定的对象。此方法返回此集合中指定索引位置的原元素                                                                         |
| `List<E> subList(int fromlndex, int tolndex)` | 返回一个新的集合，新集合中包含 fromlndex 和 tolndex 索引之间的所有元素。包含 fromlndex 处的元素，不包含 tolndex 索引处的元素 `LinkedList`包含Collection方法，还有特有方法 |
                                                                                                                       |
| 方法名                                        | 说明                                                                                                                                                                      |
| -------------------------                     | --------------------------------                                                                                                                                          |
| public void addFirst(E e)                     | 在该列表开头插入指定的元素                                                                                                                                                |
| public void addLast(E e)                      | 将指定的元素追加到此列表的末尾                                                                                                                                            |
| public E getFirst()                           | 返回此列表中的第一个元素                                                                                                                                                  |
| public E getLast()                            | 返回此列表中的最后一个元素                                                                                                                                                |
| public E removeFirst()                        | 从此列表中删除并返回第一个元素                                                                                                                                            |
| public E removeLast()                         | 从此列表中删除并返回最后一个元素                                                                                                                                          |

#### 5.1.2 list下安全类CopyOnWriteArrayList

CopyOnWriteArrayList
**基本原理：**

**add()方法**其实他会**加lock锁**，然后会**复制出一个新的数组**，往新的数组里边add真正的元素，最后把array的指向改变为新的数组

**get()方法**又或是size()方法只是获取array所指向的数组的元素或者大小。**读不加锁**，写加锁



**优缺点**：

读操作性能很高，因为无需任何同步措施，比较适用于读多写少的并发场景

读写分离，提高了读的效率



每次执行写操作都要将原容器拷贝一份，内存占用大

只能保证数据的最终一致性，**不能保证数据的实时一致性**。假设两个线程，线程A去读取CopyOnWriteArrayList的数据，还没读完，现在线程B把这个List给清空了，线程A此时还是可以把剩余的数据给读出来。

#### 5.1.3 vector

Vector是**底层结构是数组**，一般现在我们已经很少用了。

相对于ArrayList，它是线程安全的，在扩容的时候它是直接**扩容两倍**的

比如现在有10个元素，要扩容的时候，就会将数组的大小增长到20



### 5.2 Set集合

- Set集合的特点
    - 元素存取无序
    - 没有索引、只能通过迭代器或增强for循环遍历
    - 不能存储重复元素
- 哈希值【理解】
    - 哈希值简介
        - 是JDK根据对象的地址或者字符串或者数字算出来的int类型的数值
    - 如何获取哈希值
        - Object类中的public int hashCode()：返回对象的哈希码值
    - 哈希值的特点
        - 同一个对象多次调用hashCode()方法返回的哈希值是相同的
        - 默认情况下，不同对象的哈希值是不同的。而重写hashCode()方法，可以实现让不同对象的哈希值相同
        - 在需要重写`equals()`方法时，也同时需要重写`hashCode()`方法
- HashSet集合的特点
    - 底层数据结构是哈希表
    - 对集合的迭代顺序不作任何保证，也就是说不保证存储和取出的元素顺序一致
    - 没有带索引的方法，所以不能使用普通for循环遍历
    - 由于是Set集合，所以是不包含重复元素的集合
    - 不同步的，多线程访问需要设置同步
- HashSet集合保证元素唯一性

![image](https://note.youdao.com/yws/public/resource/9a17bf7311b3424a6840aae93c5a8648/xmlnote/01190DAD843A4320950BE50559F86927/19474)
- 数据结构之哈希表
![image](https://note.youdao.com/yws/public/resource/9a17bf7311b3424a6840aae93c5a8648/xmlnote/4EBC952395EA4AA585FB91B7E7C448C3/19478)

- LinkedHashSet集合特点
    - 哈希表和链表实现的Set接口，具有可预测的迭代次序
    - 由链表保证元素有序，也就是说元素的存储和取出顺序是一致的
    - 由哈希表保证元素唯一，也就是说没有重复的元素
    
- TreeSet集合概述

    - 只能添加**同一类型对象**，如不能同时添加`String`和`Date`类型
    - 元素有序，可以按照一定的规则进行排序，具体排序方式取决于构造方法
        - TreeSet()：根据其元素的自然排序进行排序
        - TreeSet(Comparator comparator) ：根据指定的比较器进行排序
    - 没有带索引的方法，所以不能使用普通for循环遍历
    - 由于是Set集合，所以不包含重复元素的集合

    ```java
    	// TreeSet集合基本方法使用        
    		TreeSet<Object> ts = new TreeSet<>();
            ts.add(10);
            ts.add(2);
            ts.add(-9);
            ts.add(5);
            System.out.println(ts);  // [-9, 2, 5, 10]
            System.out.println(ts.first()); //-9
            System.out.println(ts.last());  // 10
            System.out.println(ts.headSet(5));  // [-9, 2]
            System.out.println(ts.tailSet(5));  // [5, 10]
            System.out.println(ts.subSet(0,10));  // [2, 5]
    ```

    
```java
// 方法一： 在定义学生类时实现Comparable类并重写compareTo方法实现按照比较器排序
public class Student implements Comparable<Student> {
    private int age;
    
    @Override
    public int compareTo(Student s) {
// return 0;  返回0则不比较
// return 1;
// return -1;
//按照年龄从小到大排序
    int num = this.age - s.age;  this.age代表后面的值，从小到大排序
// int num = s.age - this.age;
//年龄相同时，按照姓名的字母顺序排序
    int num2 = num==0?this.name.compareTo(s.name):num;
    return num2;
    }
}
```

```java
// 方法二： 就是让集合构造方法接收Comparator的实现类对象，重写compare(T o1,T o2)方法
public static void main(String[] args) {
    //创建集合对象时，在构造方法中通过匿名类重写compare方法
    TreeSet<Student> ts = new TreeSet<Student>( new Comparator<Student>() {
    @Override
    public int compare(Student s1, Student s2) {
    //this.age - s.age
    //s1,s2
    int num = s1.getAge() - s2.getAge();
    int num2 = num == 0 ? s1.getName().compareTo(s2.getName()): num;
    return num2;
        }
    });

...
}

```

### 5.3 Map

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/Java/008i3skNgy1gtv61zpemjj61h80mgqai02.jpg)



- Map集合的特点
    - 键值对映射关系
    - 一个键对应一个值
    - 键不能重复，值可以重复
    - 元素存取无序
    - HashMap和TreeMap子类

| 方法名                              | 说明                                 |
| ----------------------------------- | ------------------------------------ |
| V put(K key,V value)                | 添加元素                             |
| V remove(Object key)                | 根据键删除键值对元素                 |
| void clear()                        | 移除所有的键值对元素                 |
| boolean containsKey(Object key)     | 判断集合是否包含指定的键             |
| boolean containsValue(Object value) | 判断集合是否包含指定的值             |
| boolean isEmpty()                   | 判断集合是否为空                     |
| int size()                          | 集合的长度，也就是集合中键值对的个数 |

| 获取的相关方法                             | 说明                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| V get(Object key)                          | 根据键获取值                                                 |
| Set keySet()                               | 获取所有键的集合                                             |
| Collection values()                        | 获取所有值的集合                                             |
| Set<Map.Entry<K,V>> entrySet()             | 获取所有键值对对象的集合                                     |
| `getOrDefault(Object key, V defaultValue)` | 返回值指定的键映射,或者  `defaultValue`如果这张Map不包含映射的关键。 |

```java
// 常用的遍历Map集合方法：
// 方法一：通过keySet获取所有键，根据get获取相应的值
Set<String> keySet = map.keySet();
//遍历键的集合，获取到每一个键。用增强for实现
for (String key : keySet) {
//根据键去找值。用get(Object key)方法实现
    String value = map.get(key);
    System.out.println(key + "," + value);
}
// 方法二：通过entrySet()获取键值对
Set<Map.Entry<String, String>> entrySet = map.entrySet();
//遍历键值对对象的集合，得到每一个键值对对象
for (Map.Entry<String, String> me : entrySet) {
//根据键值对对象获取键和值
    String key = me.getKey();
    String value = me.getValue();
    System.out.println(key + "," + value);
}

```



#### ConcurrentHashMap

jdk1.8原理：对每个节点`Node<K,V> f`使用`synchronized (f)`加锁，使用的是CAS锁

**（1）你知道 HashMap 的工作原理吗？你知道 HashMap 的 get() 方法的工作原理吗？**

HashMap 是基于 hashing 的原理，我们使用 put(key, value) 存储对象到 HashMap 中，使用 get(key) 从 HashMap 中获取对象。当我们给 put() 方法传递键和值时，我们先对键调用 hashCode() 方法，返回的 hashCode 用于找到 bucket 位置来储存 Entry 对象。

**（2）你知道 ConcurrentHashMap 的工作原理吗？你知道 ConcurrentHashMap 在 JAVA8 和 JAVA7 对比有哪些不同呢?**

ConcurrentHashMap 为了提高本身的并发能力，在内部采用了一个叫做 Segment 的结构，一个 Segment 其实就是一个类 Hash Table 的结构，Segment 内部维护了一个链表数组，我们用下面这一幅图来看下 ConcurrentHashMap 的内部结构,从下面的结构我们可以了解到，ConcurrentHashMap 定位一个元素的过程需要进行两次Hash操作，第一次 Hash 定位到 Segment，第二次 Hash 定位到元素所在的链表的头部，因此，这一种结构的带来的副作用是 Hash 的过程要比普通的 HashMap 要长，但是带来的好处是写操作的时候可以只对元素所在的 Segment 进行操作即可，不会影响到其他的 Segment，这样，在最理想的情况下，ConcurrentHashMap 可以最高同时支持 Segment 数量大小的写操作（刚好这些写操作都非常平均地分布在所有的 Segment上），所以，通过这一种结构，ConcurrentHashMap 的并发能力可以大大的提高。

JAVA7之前ConcurrentHashMap主要采用锁机制，在对某个Segment进行操作时，将该Segment锁定，不允许对其进行非查询操作，而在JAVA8之后采用CAS无锁算法，这种乐观操作在完成前进行判断，如果符合预期结果才给予执行，对并发操作提供良好的优化

**（3）当两个对象的hashcode相同会发生什么？**

因为hashcode相同，所以它们的bucket位置相同，‘碰撞’会发生。因为Map使用LinkedList存储对象，这个Entry（包含有键值对的Map.Entry对象）会存储在LinkedList中。（当向 Map 中添加 key-value 对，由其 key 的 hashCode() 返回值决定该 key-value 对（就是 Entry 对象）的存储位置。当两个 Entry 对象的 key 的 hashCode() 返回值相同时，将由 key 通过 eqauls() 比较值决定是采用覆盖行为（返回 true），还是产生 Entry 链（返回 false）），此时若你能讲解JDK1.8红黑树引入，面试官或许会刮目相看。

**（4）如果两个键的 hashcode 相同，你如何获取值对象？**

当我们调用get()方法，HashMap 会使用键对象的 hashcode 找到 bucket 位置，然后获取值对象。如果有两个值对象储存在同一个 bucket，将会遍历 LinkedList 直到找到值对象。找到 bucket 位置之后，会调用 keys.equals() 方法去找到 LinkedList 中正确的节点，最终找到要找的值对象。（当程序通过 key 取出对应 value 时，系统只要先计算出该 key 的 hashCode() 返回值，在根据该 hashCode 返回值找出该 key 在 table 数组中的索引，然后取出该索引处的 Entry，最后返回该 key 对应的 value 即可）。

**（5）如果HashMap的大小超过了负载因子（load factor）定义的容量，怎么办？**

当一个map填满了75%的bucket时候，和其它集合类（如ArrayList等）一样，将会创建原来HashMap大小的两倍的bucket数组，来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫作rehashing，因为它调用hash方法找到新的bucket位置。

**（6）你了解重新调整HashMap大小存在什么问题吗？**

当重新调整HashMap大小的时候，确实存在条件竞争，因为如果两个线程都发现HashMap需要重新调整大小了，它们会同时试着调整大小。在调整大小的过程中，存储在LinkedList中的元素的次序会反过来，因为移动到新的bucket位置的时候，HashMap并不会将元素放在LinkedList的尾部，而是放在头部，这是为了避免尾部遍历(tail traversing)。如果条件竞争发生了，那么就死循环了。这个时候，你可以质问面试官，为什么这么奇怪，要在多线程的环境下使用HashMap呢？



**1. ConcurrentHashMap中变量使用final和volatile修饰有什么用呢？**
Final域使得确保初始化安全性（initialization safety）成为可能，初始化安全性让不可变形对象不需要同步就能自由地被访问和共享。
使用volatile来保证某个变量内存的改变对其他线程即时可见，在配合CAS可以实现不加锁对并发操作的支持。get操作可以无锁是由于Node的元素val和指针next是用volatile修饰的，在多线程环境下线程A修改结点的val或者新增节点的时候是对线程B可见的。

**2.我们可以使用CocurrentHashMap来代替Hashtable吗？**
我们知道Hashtable是synchronized的，但是ConcurrentHashMap同步性能更好，因为它仅仅根据同步级别对map的一部分进行上锁。ConcurrentHashMap当然可以代替HashTable，但是HashTable提供更强的线程安全性。它们都可以用于多线程的环境，但是当Hashtable的大小增加到一定的时候，性能会急剧下降，因为迭代时需要被锁定很长的时间。因为ConcurrentHashMap引入了分割(segmentation)，不论它变得多么大，仅仅需要锁定map的某个部分，而其它的线程不需要等到迭代完成才能访问map。简而言之，在迭代的过程中，ConcurrentHashMap仅仅锁定map的某个部分，而Hashtable则会锁定整个map。

**3. ConcurrentHashMap有什么缺陷吗？**
ConcurrentHashMap 是设计为非阻塞的。在更新时会局部锁住某部分数据，但不会把整个表都锁住。同步读取操作则是完全非阻塞的。好处是在保证合理的同步前提下，效率很高。**坏处是严格来说读取操作不能保证反映最近的更新**。例如线程A调用putAll写入大量数据，期间线程B调用get，则只能get到目前为止已经顺利插入的部分数据。

**4. ConcurrentHashMap在JDK 7和8之间的区别**

- JDK1.8的实现降低锁的粒度，JDK1.7版本锁的粒度是基于Segment的，包含多个HashEntry，而JDK1.8锁的粒度就是HashEntry（首节点）
- JDK1.8版本的数据结构变得更加简单，使得操作也更加清晰流畅，因为已经使用synchronized来进行同步，所以不需要分段锁的概念，也就不需要Segment这种数据结构了，由于粒度的降低，实现的复杂度也增加了
- JDK1.8使用红黑树来优化链表，基于长度很长的链表的遍历是一个很漫长的过程，而红黑树的遍历效率是很快的，代替一定阈值的链表，这样形成一个最佳拍档



### 5.4 集合嵌套

```java
// ArrayList中嵌套HashMap
ArrayList< HashMap<String, String> > array = new ArrayList< HashMap<String, String> >();

// HashMap中嵌套ArrayList
HashMap<String, ArrayList<String>> hm = new HashMap<String, ArrayList<String>>();
```

### 5.5 Collections集合工具类
| 方法名                                   | 说明                               |
| ---------------------------------------- | ---------------------------------- |
| public static void sort(List list)       | 将指定的列表按升序排序             |
| public static void reverse(List<?> list) | 反转指定列表中元素的顺序           |
| public static void shuffle(List<?> list) | 使用默认的随机源随机排列指定的列表 |



## 队列

### PriorityQueue

自动排序的队列，数字按从小到大排序，可以通过定制Comparetor定制排序

| offer() | 添加元素，比add()更好                             |
| ------- | ------------------------------------------------- |
| poll()  | 删除队列头部元素，为空返回null，比remove更好      |
| peek()  | 显示对头元素，不删除，为空返回null，比element更好 |
|         |                                                   |



```java
        PriorityQueue<Integer> pq = new PriorityQueue<>();
        pq.offer(10);
        pq.offer(3);
        pq.offer(11);
        System.out.println("pq : " + pq); // pq : [3, 10, 11]
        System.out.println("pq.poll() : " + pq.poll());  // pq.poll() : 3
        while(pq.poll() != null){  // 此处如果替换为pq.remove()则会出错
            System.out.println(pq);  // [11]  // []
        }
```



### ArrayDeque



```java
        ArrayDeque<Object> stack = new ArrayDeque<>();
        stack.push(9);
        stack.push("a");
        stack.push(-1);
        System.out.println(stack); // [-1, a, 9]
        System.out.println(stack.pop()); // -1
        System.out.println(stack.peek());  // a
```







## 6. 泛型

- 含义： 就是将类型由原来的具体的类型参数化，然后在使用/调用时传入具体的类型
- 使用： 这种参数类型可以用在类、方法和接口中，分别被称为泛型类、泛型方法、泛型接口

```java
// 1. 泛型类定义
public class Generic<T> {
    private T t;
    
    public T getT() {
        return t;
    }
    public void setT(T t) {
        this.t = t;
    }
}
// 实例化泛型
Generic<String> g1 = new Generic<String>();
Generic<Integer> g2 = new Generic<Integer>();

// 2. 泛型方法定义
public <T> void show(T t) {
    System.out.println(t);
}
g.show(30);
g.show(true);



// 3. 泛型接口定义
public interface Generic<T> {
    void show(T t);
}

public class GenericImpl<T> implements Generic<T> {
    @Override
    public void show(T t) {
        System.out.println(t);
    }
}
Generic<String> g1 = new GenericImpl<String>();
g1.show("林青霞");

```
## 7. 类型通配符

- 作用：表示各种泛型List的父类
- 类型通配符：`<?>`
	- `List<?>`：表示元素类型未知的List，它的元素可以匹配任何的类型
	- 这种带通配符的List仅表示它是各种泛型List的父类，并不能把元素添加到其中
- 类型通配符上限：`<? extends 类型>`
- 类型通配符下限：`<? super 类型>`
    - 例如：`List<? super Number>`：它表示的类型是Number或者其父类型`Object`
    - 例如： `List<? extends Number>`：它表示的类型是Number或者其子类型`Integer`



## 8. 可变参数
`int... a`表示，多个变量时可变参数放最后：`int b, int... a`

```java
public static int sum(int... a) {
    int sum = 0;
    for(int i : a) { sum += i; }
    return sum;
}
```

## 9. IO文件

### 9.1 File

#### 9.1.1 File基本介绍声明
- File类介绍
    - 它是文件和目录路径名的抽象表示
    - 文件和目录是可以通过File封装成对象的
    - 对于File而言，其封装的并不是一个真正存在的文件，仅仅是一个路径名而已。它可以是存在的，==也可以是不存在的==。将来是要通过具体的操作把这个路径的内容转换为具体存在的
    
| 方法名                            | 说明                                                        |
| --------------------------------- | ----------------------------------------------------------- |
| File(String pathname)             | 通过将给定的路径名字符串转换为抽象路径名来创建新的 File实例 |
| File(String parent, String child) | 从父路径名字符串和子路径名字符串创建新的File实例            |
| File(File parent, String child)   | 从父抽象路径名和子路径名字符串创建新的 File实例             |

示例：`File f1 = new File("E:\\itcast\\java.txt");`

#### 9.1.2 常用创建功能方法
| 方法名                         | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| public boolean createNewFile() | 当具有该名称的文件不存在时，创建一个由该抽象路径名命名的新空文件 |
| public boolean mkdir()         | 创建由此抽象路径名命名的目录                                 |
| public boolean mkdirs()        | （创建多级不存在目录）创建由此抽象路径名命名的目录，包括任何必需但不存在的父目录 |

`File f4 = new File("E:\\itcast\\javase.txt");System.out.println(f4.mkdir());`

此时能够创建一个文件夹为`javase.txt`的目录

#### 9.1.3 判断和获取功能
- 判断功能

| 方法名                       | 说明                                 |
| ---------------------------- | ------------------------------------ |
| public boolean isDirectory() | 测试此抽象路径名表示的File是否为目录 |
| public boolean isFile()      | 测试此抽象路径名表示的File是否为文件 |
| public boolean exists()      | 测试此抽象路径名表示的File是否存在   |

- 获取功能

| 方法名                          | 说明                                                     |
| ------------------------------- | -------------------------------------------------------- |
| public String getAbsolutePath() | 返回此抽象路径名的绝对路径名字符串                       |
| public String getPath()         | 将此抽象路径名转换为路径名字符串                         |
| public String getName()         | 返回由此抽象路径名表示的文件或目录的名称                 |
| public String[] list()          | 返回此抽象路径名表示的目录中的文件和目录的名称字符串数组 |
| public File[] listFiles()       | 返回此抽象路径名表示的目录中的文件和目录的File对象数组   |

- 删除功能

`public boolean delete() `删除由此抽象路径名表示的文件或目录

#### 9.1.4 递归遍历输出当前目录下所有的文件名
```java
public static void getAllFileName(File f){
        File[] filelist = f.listFiles();  // 获取当前目录列表
        if(filelist != null){
            for(File e: filelist){
                if(e.isDirectory() ){  // 当当前目录下还含有文件夹的话继续递归遍历
                    getAllFileName(e);
                }else{
                    System.out.println(e.getName());  // 获取当前目录下的文件名称
                }
            }
        }
    }
```


### 9.2 I/O

![image](https://note.youdao.com/yws/public/resource/9a17bf7311b3424a6840aae93c5a8648/xmlnote/37C9590457C443619F5B19FB6C35781F/19903)
![image](https://note.youdao.com/yws/public/resource/9a17bf7311b3424a6840aae93c5a8648/xmlnote/C5BDC1ADEF884B9BA33FD739A4B26ED7/19904)

#### 9.2.1 常用方法
`FileOutputStream`常用方法，将指定字节写入文件中

| 方法名                                 | 说明                                                         |
| -------------------------------------- | ------------------------------------------------------------ |
| void write(int b)                      | 将指定的字节写入此文件输出流 一次写一个字节数据              |
| void write(byte[] b)                   | 将b.length字节从指定的字节数组写入此文件输出流一次写一个字节数组数据 |
| void write(byte[] b, int off, int len) | 将len字节从指定的字节数组开始，从偏移量off开始写入此文件输出流 一次写一个字节数组的部分数据 |
| void close()                           | 关闭数据流，当完成对数据流的操作之后需要关闭数据流           |

默认是覆盖源文件重新写入，如果源文件不存在则创建该文件并写入

通过`FileOutputStream fos = new FileOutputStream("myByteStream\\fos.txt",true);`在后面追加`true`后则为追加写入，而非覆盖源文件。


`FileInputStream`常用方法
| 名称                               | 作用                                                         |
| :--------------------------------- | :----------------------------------------------------------- |
| int read()                         | 从输入流读入一个 8 字节的数据，将它转换成一个 0~ 255的整数，返回一个整数，如果遇到输入流的结尾返回 -1 |
| int read(byte[] b)                 | 从输入流读取若干字节的数据保存到参数 b 指定的字节数组中，返回的字节数表示读取的字节数，如果遇到输入流的结尾返回 -1 |
| int read(byte[] b,int off,int len) | 从输入流读取若干字节的数据保存到参数 b 指定的字节数组中，其中 off 是指在数组中开始保存数据位置的起始下标，len 是指读取字节的位数。返回的是实际读取的字节数，如果遇到输入流的结尾则返回 -1 |
| void close()                       | 关闭数据流，当完成对数据流的操作之后需要关闭数据流           |

在==用完文件资源后一定要释放资源==：`fos.close()`或`fis.close()`

#### 9.2.2 异常处理
```java
FileOutputStream fos = null;
try {
    fos = new FileOutputStream("myByteStream\\fos.txt");
    fos.write("hello".getBytes());
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if(fos != null) {
        try {
            fos.close();   
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

#### 9.2.3 字节流读数据

1. 一次读一个字节数据
```java
    FileInputStream fis = new FileInputStream("myByteStream\\fos.txt");
    int by;
    /*
    fis.read()：读数据
    by=fis.read()：把读取到的数据赋值给by
    by != -1：判断读取到的数据是否是-1
    */
    while ((by=fis.read())!=-1) {
        System.out.print((char)by);
    }
    //释放资源
    fis.close();
```
2. 一次读一个字节数组数据，复制文件
```java
//根据数据源创建字节输入流对象
FileInputStream fis = new FileInputStream("E:\\itcast\\mn.jpg");
//根据目的地创建字节输出流对象
FileOutputStream fos = new FileOutputStream("myByteStream\\mn.jpg");
//读写数据，复制图片(一次读取一个字节数组，一次写入一个字节数组), 一般大小为1024*n
byte[] bys = new byte[1024];  
int len;
while ((len=fis.read(bys))!=-1) {
    fos.write(bys,0,len);
}
//释放资源
fos.close();
fis.close();
```

## 10. 字节字符缓冲流

### 10.1 字节缓冲流

- 字节缓冲流介绍
    - lBufferOutputStream：该类实现缓冲输出流。 通过设置这样的输出流，应用程序可以向底层输出流写入字节，而不必为写入的每个字节导致底层系统的调用
    - lBufferedInputStream：创建BufferedInputStream将创建一个内部缓冲区数组。 当从流中读取或跳过字节时，内部缓冲区将根据需要从所包含的输入流中重新填充，一次很多字节
    
| 方法名                                 | 说明                   |
| -------------------------------------- | ---------------------- |
| BufferedOutputStream(OutputStream out) | 创建字节缓冲输出流对象 |
| BufferedInputStream(InputStream in)    | 创建字节缓冲输入流对象 |

- 通过字节数组缓冲流复制数据：这种方式传输最快，相对单字节的缓冲流和字节数组的字节流来说
```java
    BufferedInputStream bis = new BufferedInputStream( new FileInputStream("D:\\temp\\a.exe") );
    BufferedOutputStream bos = new BufferedOutputStream( new FileOutputStream("D:\\temp\\a4.exe") );

    int len;
    byte[] bys = new byte[1024];
    while ( (len = bis.read(bys)) != -1 ){
        bos.write(bys, 0, len);
    }

    bos.close();
    bis.close();
```

### 10.2 字符流

#### 10.2.1 基本介绍
- 字符流的介绍
    - 由于字节流操作中文不是特别的方便，所以Java就提供字符流
    - 字符流 = 字节流 + 编码表
- 编码表
    - 什么是字符集：字符编码，有ASCII字符集、GBXXX字符集、Unicode字符集等 


字符串中的编码解码问题
| 方法名                                   | 说明                                               |
| ---------------------------------------- | -------------------------------------------------- |
| byte[] getBytes()                        | 使用平台的默认字符集将该 String编码为一系列字节    |
| byte[] getBytes(String charsetName)      | 使用指定的字符集将该 String编码为一系列字节        |
| String(byte[] bytes)                     | 使用平台的默认字符集解码指定的字节数组来创建字符串 |
| String(byte[] bytes, String charsetName) | 通过指定的字符集解码指定的字节数组来创建字符串     |

```java
    String a = "中国boy";

    byte[] bysUtf = a.getBytes();   // 默认UTF-8
    byte[] bysGbk = a.getBytes("GBK");  // 通过GBK编码，得到字节数组
    System.out.println(Arrays.toString(bysUtf)); // 输出字节数组
    System.out.println(Arrays.toString(bysGbk));

    String aUtf = new String(bysUtf,"UTF-8");  // 按照UTF-8格式解码成字符
    String aGbk = new String( bysGbk, "GBK" );
```

- 字节流是否可以取代字符流？字符流存在的意义？
    - 当对非英文字符操作时，读取文本操作时对于多种编码格式的文本非常有必要使用字符流
    - 如果对不同编码格式的文本仅仅是复制传输操作，而不进行识别后操作的话没有必要使用字符流，反而会增加编码解码时间
    - 例如读取中文文本后，将文本写入数据集中数据库中，将文本显示在输出框中则很有必要
    - 字符流的`BufferedReader`和`BufferedWriter`有一个特有的方法`readline`and`nextline`仅针对`UTF-8`默认格式的存在（因为只能通过`FileWriter`和`FileReader`声明，而它们只能使用默认格式）

#### 10.2.2 字符流中的编码解码问题
- InputStreamReader：它读取字节，并使用指定的编码将其解码为字符
- OutputStreamWriter：是从字符流到字节流的桥梁，使用指定的编码将写入的字符编码为字节
```java
    OutputStreamWriter osw2 = new OutputStreamWriter( new FileOutputStream("idea_demo\\java2.txt") ,"GBK2312");
    InputStreamReader isr2 = new InputStreamReader( new FileInputStream("idea_demo\\java.txt") ,"GBK2312");

    int len2;
    char[] chs2 = new char[1024];
    while( (len2 = isr2.read(chs2)) != -1 ){
        // osw2.write( new String(chs2, 0, len2));
        osw2.write( chs2, 0, len2 )
    }

    osw2.close();
    isr2.close();
```
| 方法名                                             | 说明                                         |
| -------------------------------------------------- | -------------------------------------------- |
| InputStreamReader(InputStream in)                  | 使用默认字符编码创建InputStreamReader对象    |
| InputStreamReader(InputStream in,String chatset)   | 使用指定的字符编码创建InputStreamReader对象  |
| OutputStreamWriter(OutputStream out)               | 使用默认字符编码创建OutputStreamWriter对象   |
| OutputStreamWriter(OutputStream out,Stringcharset) | 使用指定的字符编码创建OutputStreamWriter对象 |

- 字符流写数据的5种方式

| 方法名                                    | 说明                 |
| ----------------------------------------- | -------------------- |
| void write(int c)                         | 写一个字符           |
| void write(char[] cbuf)                   | 写入一个字符数组     |
| void write(char[] cbuf, int off, int len) | 写入字符数组的一部分 |
| void write(String str)                    | 写一个字符串         |
| void write(String str, int off, int len)  | 写一个字符串的一部分 |


- `close()` 关闭流，释放资源，但是在关闭之前会先刷新流。一旦关闭，就不能再写数据

- 字符流读数据

| 方法名                | 说明                   |
| --------------------- | ---------------------- |
| int read()            | 一次读一个字符数据     |
| int read(char[] cbuf) | 一次读一个字符数组数据 |


#### 10.2.3 字符缓冲流

- BufferedWriter：将文本写入字符输出流，缓冲字符，以提供单个字符，数组和字符串的高效写入，可以指定缓冲区大小，或者可以接受默认大小。默认值足够大，可用于大多数用途
- BufferedReader：从字符输入流读取文本，缓冲字符，以提供字符，数组和行的高效读取，可以指定缓冲区大小，或者可以使用默认大小。 默认值足够大，可用于大多数用途

| 方法名                     | 说明                   |
| -------------------------- | ---------------------- |
| BufferedWriter(Writer out) | 创建字符缓冲输出流对象 |
| BufferedReader(Reader in)  | 创建字符缓冲输入流对象 |

- `BufferedWriter`特有方法：`void newLine()` 写一行行分隔符，行分隔符字符串由系统属性定义
- `BufferedReader`特有方法`String readLine()` 读一行文字。结果包含行的内容的字符串，不包括任何行终止字符如果流的结尾已经到达，则为`null`

```java
    BufferedReader br = new BufferedReader( new FileReader("D:\\temp\\aaa.txt") );
    BufferedWriter bw = new BufferedWriter( new FileWriter("D:\\temp\\aaa4.txt") );

    String line;
    int lens;
    while( (line = br.readLine()) != null ){
        bw.write(line2);
        bw.newLine();
        bw.flush();
    }
    
//  int blen;
//  char[] bchs = new char[1024];
//  while( (blen = br.read(bchs)) != -1){
//      bw.write( new String(bchs, 0 ,blen) );
//  }

    br.close();
    bw.close();
```

## 11. 特殊操作流

### 11.1 `Scanner`和`BufferedReader`的区别
- `Scanner`:解析基本数据类型和字符串。它本质上是使用正则表达式去读取不同的数据类型
- `BufferedReader`: 高效的读取字符序列，从字符输入流和字符缓冲区读取文本，读取固定一行`BufferedReader br = new BufferedReader (newInputStreamReader(System.in))`，返回字符串
- 两者对比
    1. `Scanner`提供了一系列`nextXxx()`方法，当我们确定输入的数据类型时，使用`Scanner`更加方便。也正是因为这个`BufferedReader`相对于`Scanner`来说要快一点，因为`Scanner`对输入数据进行类解析，而`BufferedReader`只是简单地读取字符序列。
    2. `Scanner`和`BufferedReader`都设置了缓冲区，`Scanner`有很少的缓冲区(1KB字符缓冲)相对于`BufferedReader`(8KB字节缓冲)，但是这是绰绰有余的。
    3. `BufferedReader`是支持同步的，而`Scanner`不支持。如果我们处理多线程程序，`BufferedReader`应当使用。
    4. `Scanner`输入的一个问题：在`Scanner`类中如果我们在任何7个nextXXX()方法之后调用`nextLine()`方法，这`nextLine()`方法不能够从控制台读取任何内容，并且，这游标不会进入控制台，它将跳过这一步。`nextXXX()`方法包括`nextInt()，nextFloat()， nextByte()，nextShort()，nextDouble()，nextLong()，next()`。在`BufferReader`类中就没有那种问题。这种问题仅仅出现在`Scanner`类中，由于`nextXXX()`方法忽略换行符，但是`nextLine()`并不忽略它。如果我们`在nextXXX()`方法和`nextLine()`方法之间使用超过一个以上的`nextLine()`方法，这个问题将不会出现了；因为`nextLine()`把换行符消耗了。

原文链接：https://blog.csdn.net/zengxiantao1994/article/details/78056243

```java
    // 通过BufferedReader方式读取数据
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    System.out.println(br.readLine());
    
    // 通过系统提供的Scanner方式
    Scanner sc = new Scanner(System.in);
    System.out.println(sc.nextInt());
    sc.nextLine();    // 必须要消耗一个nextLine，否则当输入第一个数字后按enter换行时直接结束，消耗了一个换行 
    System.out.println(sc.nextLine());
```

### 11.2 标准输入流

`public static final InputStream in` 用户键盘主机输入

`InputStream is = System.in`

`Scanner(InputStream source)`是Scanner的一种构造方法，使用的`Scanner(System.in)`

### 11.3 标准输出流
`PrintStream ps = System.out;`

`PrintStream`类的所有方法，`System.out`都可以使用，例如：`print()`/`printf()`/`println()`

### 11.4 字符打印流

| 方法名                                     | 说明                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| PrintWriter(String fileName)               | 使用指定的文件名创建一个新的PrintWriter，而不需要自动执行刷新 |
| PrintWriter(Writer out, boolean autoFlush) | 创建一个新的PrintWriter out：字符输出流 autoFlush： 一个布尔值，如果为真，则println，printf，或format方法将刷新输出缓冲区 |

```java
    PrintWriter pw2 = new PrintWriter( new FileWriter("idea_demo\\java3.txt", true), true);   
    // 第一个true为添加写入文件，第二个true为自动刷新无需手动写pw.flush
    pw2.write("hellp");
    pw2.println(99); // 直接写入99
    pw2.write(99);  // 转换为字符 c 写入
    pw2.close();
```

### 11.5 对象序列化流

- 对象序列化介绍
    - 对象序列化：就是将对象保存到磁盘中，或者在网络中传输对象
    - 这种机制就是使用一个字节序列表示一个对象，该字节序列包含：对象的类型、对象的数据和对象中存储的属性等信息
    - 字节序列写到文件之后，相当于文件中持久保存了一个对象的信息
    - 反之，该字节序列还可以从文件中读取回来，重构对象，对它进行反序列化
- 对象序列化流： ObjectOutputStream
    - 将Java对象的原始数据类型和图形写入OutputStream。
    - 可以使用ObjectInputStream读取（重构）对象。
    - 可以通过使用流的文件来实现对象的持久存储。
    - 如果流是网络套接字流，则可以在另一个主机上或另一个进程中重构对象
- 构造方法

| 方法名                                | 说明                                               |
| ------------------------------------- | -------------------------------------------------- |
| Object OutputStream(OutputStream out) | 创建一个写入指定的OutputStream的ObjectOutputStream |
-  序列化对象的方法：`void writeObject(Object obj)`
- 对象反序列化：`Object InputStream(InputStream in)`创建从指定的`InputStream`读取的`ObjectInputStream`
- 对象反序列化方法:`Object readObject()` 从`ObjectInputStream`读取一个对象

```java
// 对象要实现Serializable接口以表示该类使用了对象序列化
    public class Student implements Serializable {
        private static final long serialVersionUID = 42L; 
        // 设定serialVersionUID为固定值，后面如果该对象发生改变也可以继续读取，否则如果该对象发生了一点点变化也无法读取成功，建议设定固定值
        private String name;
        private transient int age;  // 通过transient使得该变量不参与对象序列化
        public Student() {
        }
        ......
    }

    // 写入对象序列
    ObjectOutputStream oos = new ObjectOutputStream( new FileOutputStream("idea_demo\\java4.txt") );
    Student s1 = new Student("中国", 10,20,30);
    Student s2 = new Student("中国11", 10,20,32);
    oos.writeObject(s1);
    oos.writeObject(s2);
    oos.close();
    
    // 读取多个对象序列
    ObjectInputStream ois = new ObjectInputStream( new FileInputStream("idea_demo\\java4.txt") );
    Object obj = null;
    while (true){    // 读取多个对象
        try{
            Student ss = (Student)ois.readObject();
            System.out.println(ss.getName() + ":" + ss.getAge());  // 对于没有序列化的数据默认为0
        }catch (EOFException e){
            break;
        }catch (NullPointerException ee){
            continue;
        }
    }
    ois.close();
```

## 12 Properties集合

- Properties类继承自HashTable，通常和io流结合使用。
- 它最突出的特点是将key/value作为配置属性写入到配置文件中以实现配置持久化，或从配置文件中读取这些属性。
- 它的这些配置文件的规范后缀名为".properties"。表示了一个持久的属性集。

| 方法名                                        | 说明                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| put(Object key, Object key)                   | 将指定的 key映射到此散列表中指定的 value                     |
| get(Object key)                               | 返回指定键映射到的值，如果此映射不包含键的映射，则返回 null  |
| Object setProperty(String key,String value)   | 设置集合的键和值，都是String类型，底层调用 Hashtable方法     |
| put String getProperty(String key)            | 使用此属性列表中指定的键搜索属性                             |
| Set stringPropertyNames()                     | 从该属性列表中返回一个不可修改的键集，其中键及其对应的值是字符串 |
| void load(InputStream inStream)               | 从输入字节流读取属性列表（键和元素对）                       |
| void load(Reader reader)                      | 从输入字符流读取属性列表（键和元素对）                       |
| void store(OutputStream out, String comments) | 将此属性列表（键和元素对）写入此 Properties表中，以适合于使用load(InputStream)方法的格式写入输出字节流 |
| void store(Writer writer, String comments)    | 将此属性列表（键和元素对）写入此Properties表中，以适合使用 load(Reader)方法的格式写入输出字符流 |

## 网络编程

### 13.1 `InetAddress`
```java
//InetAddress address = InetAddress.getByName("itheima");
InetAddress address = InetAddress.getByName("192.168.1.66");

//public String getHostName()：获取此IP地址的主机名
String name = address.getHostName();

//public String getHostAddress()：返回文本显示中的IP地址字符串
String ip = address.getHostAddress();
```

### 13.2 UDP通信
- `UDP`使用`DatagramSocket`创建接收和发送端的socket
- 使用`DatagramPacket`封装报文`DatagramPacket(byte[] buf, int length, InetAddress address, int port)`需要字节数组

```java
Send: 
    //1. 创建发送端的Socket对象(DatagramSocket)
    DatagramSocket ds = new DatagramSocket();
    
    //自己封装键盘录入数据
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    String line;
    while ((line = br.readLine()) != null) {
        //输入的数据是886，发送数据结束
        if ("886".equals(line)) {
        break;
        }
        //2. 创建数据，并把数据打包
        byte[] bys = line.getBytes();
        DatagramPacket dp = new DatagramPacket(bys, bys.length, InetAddress.getByName("192.168.1.66"), 12345);
        
        //3. 调用DatagramSocket对象的方法发送数据
        ds.send(dp);
    }
        
    //4. 关闭发送端
    ds.close();
```
```java
Receive: 
    //1. 创建接收端的Socket对象(DatagramSocket)
    DatagramSocket ds = new DatagramSocket(12345);
    
    while (true) {
        //2. 创建一个数据包，用于接收数据
        byte[] bys = new byte[1024];
        DatagramPacket dp = new DatagramPacket(bys, bys.length);
        
        //3. 调用DatagramSocket对象的方法接收数据
        ds.receive(dp);
        
        //4. 解析数据包
        System.out.println("数据是：" + new String(dp.getData(), 0,
        dp.getLength()));
    }
    ds.close()
```


### 13.3 TCP通信

- 客户端提供了`Socket`类，为服务器端提供了`ServerSocket`类并提供`accept()`方法接收`Socket`
- `Socket`类提供获取输入输出流来读取和发送数据

| 方法名                         | 说明                 |
| ------------------------------ | -------------------- |
| InputStream getInputStream()   | 返回此套接字的输入流 |
| OutputStream getOutputStream() | 返回此套接字的输出流 |

- 通过`OutputStream os = s.getOutputStream()`，然后`os.write( "haha".getByte() )`来发送数据
- 通过`InputStream is = s.getInputStream()`,然后`byte[] bys = new byte[1024]; int len = is.read(bys)`将数据读取到`bys`中
- 通过`BufferedReader br = new BufferedReader(new InputStreamReader(s.getInputStream()));`，然后使用`br.readLine()`来逐行读取字符串
- 通过`BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));`,然后使用`bw.write(String), bw.newLine()`来发送字符串

#### 13.3.1 TCP基本使用过程
```java
TCPClient
    // 1. 创建客户端
    Socket s = new Socket("10.152.137.87",2000);

    // 2. 写数据
    OutputStream os = s.getOutputStream();
    os.write("hello tcp,你好 ！".getBytes());

    // 3. 接收反馈
    InputStream is = s.getInputStream();
    byte[] bys = new byte[1024];
    int len = is.read(bys);
    System.out.println("receive data: "+ new String(bys, 0, len));

    // 4. 释放资源
    s.close();
```
```java
TCPServer
    // 1. 声明服务端套接字
    ServerSocket ss = new ServerSocket(2000);

    // 2. 接收套接字，再转为字节流
    Socket s = ss.accept();
    InputStream is = s.getInputStream();
    byte[] bys = new byte[1024];
    int len = is.read(bys);
    System.out.println("receive for client: "+ new String(bys, 0, len));

    // 3. 反馈字节流
    OutputStream os = s.getOutputStream();
    os.write("hello client啊！".getBytes());

    // 4. 释放资源
    ss.close();
```

#### 13.3.2 TCP多线程对文本复制
```java
TCPClientTread
    Socket s = new Socket("10.152.137.87",20001);
    
    // 文本字节输入输出流,一个从文本读取，一个传输给套接字以输出
    BufferedReader br = new BufferedReader(new FileReader("idea_demo\\src\\java.txt")); 
    BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));

    String line;
    while( (line = br.readLine()) != null){
    // 文本输出三件套
        bw.write(line);
        bw.newLine();
        bw.flush();
    }

    // 当文件传输结束后，通过套接字特有方法shutdownOutput来传送终止连接信号
    s.shutdownOutput();
    
    // 接收反馈
    BufferedReader brClient = new BufferedReader(new InputStreamReader(s.getInputStream()));
    System.out.println("receive from server: " + brClient.readLine());

    // 关闭套接字，释放资源
    s.close();
    br.close();
```

```java
public class TcpServerRunable implements Runnable {
    private Socket s;

    public TcpServerRunable(Socket s) {
        this.s = s;
    }

    @Override
    public void run() {
        try {
        // 定义接收写入文本的方法
            BufferedReader br = new BufferedReader(new InputStreamReader(s.getInputStream()));
            int count = 0;
            // 当文件存在时则重命名
            File f = new File("idea_demo\\src\\Network\\java[" + count + "].txt");
            while (f.exists()) {
                count++;
                f = new File("idea_demo\\src\\Network\\java[" + count + "].txt");
            }
            // 写入文本
            BufferedWriter bw = new BufferedWriter(new FileWriter(f));
            String line;
            // br.readLine逐行读取传入的字符串
            while ((line = br.readLine()) != null) {
                bw.write(line);
                bw.newLine();
                bw.flush();
            }

            BufferedWriter bwServer = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
            bwServer.write("load sucess!!!");
            bwServer.newLine();
            bwServer.flush();

            s.close();
            bw.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}


public class TcpServerThread {
    public static void main(String[] args) throws IOException {
        ServerSocket ss = new ServerSocket(20001);
        while (true){
            new Thread( new TcpServerRunable(ss.accept()) ).start();
        }
    }
}

```


## 14 Lambda

### 14.1 前提和基本使用

- 使用前提：
    - 有一个接口
    - 接口中有且仅有一个抽象方法
- 使用方式:`(参数) -> {具体使用方法}`
- 省略模式
    - 参数类型可以省略。但是有多个参数的情况下，不能只省略一个
    - 如果参数有且仅有一个，那么小括号可以省略
    - 如果代码块的语句只有一条，可以省略大括号和分号，和 return关键字
```java
public interface Eatable {
    void eat();
}

public interface ReturnAndParam {
    int sum(int a, int b, String c);
}

public interface Param {
    void eat(int a);
}

public class EatableDemo {
    public static void main(String[] args) {
    
        useEatable( () -> { System.out.println("Running away"); } );
        useEatable( () -> System.out.println("Running away") ); // 只有一句方法体时可以省略{} 
        
        useParam( (int a) -> {System.out.println("Running away"); } );
        useParam( a -> System.out.println("Running away"); ); // 只有一个参数时，可以省略()和类型
    
        useReturnAndParam( (int a,int b,String c) -> {return a+b; } );    
        useReturnAndParam( (a,b,c) -> a+b );  // 可以同时省略参数类型，只有return一句时，可以省略return和{}
        
    }    
    
    public static void useEatable(Eatable e){
        e.eat();
    }
    
    public static void useParam(Param e){
        e.eat(a);
    }
    
    public static void useReturnAndParam(ReturnAndParam e){
        int sum = e.sum(10,20,"add");
        System.out.println(sum)
    }

}
```

### 14.2 Lambda表达式和匿名内部类的区别
- 所需类型不同
    - 匿名内部类：可以是接口，也可以是抽象类，还可以是具体类
    - Lambda 表达式：只能是接口
- 使用限制不同
    - 如果接口中有且仅有一个抽象方法，可以使用 Lambda表达式，也可以使用匿名内部类
    - 如果接口中多于一个抽象方法，只能使用匿名内部类，而不能使用 Lambda表达式
- 实现原理不同
    - 匿名内部类：编译之后，产生一个单独的 .class字节码文件
    - Lambda 表达式：编译之后，没有一个单独的.class字节码文件。对应的字节码会在运行的时候动态生成

## 15 方法引用

| 种类                   | 示例               | 说明                                                         | 对应lambda表达式                       |
| ---------------------- | ------------------ | ------------------------------------------------------------ | -------------------------------------- |
| 引用类方法             | 类名::类方法       | 函数式接口中被实现方法的全部参数传给该类方法作为参数         | (a,b,..) -> 类名.类方法(a,b,...)       |
| 引用特定对象的实例方法 | 特定对象::实例方法 | 函数式接口中被实现方法的全部参数传给该方法作为参数           | (a,b,..) -> 特定对象.实例方法(a,b,...) |
| 引用某类对象的实例方法 | 类名::实例方法     | 函数式接口中被实现方法的第一个参数作为调用者，后面的参数传给该方法作为参数 | (a,b,..) -> a.实例方法(a,b,...)        |
| 引用构造器             | 类名::new          | 函数式接口中被实现方法的全部参数传给该类方法作为参数         | (a,b,..) -> new 类名(a,b,...)          |

### 15.1 引用类方法（静态方法）
```java
public interface Mytest1 {
    int test(String s);
}

//  Mytest1 mt1 = a -> Integer.parseInt(a);  lambda方式
    Mytest1 mt1 = Integer::parseInt;  // 静态方法
    Integer inta = mt1.test("100");
    System.out.println(inta);
```

### 15.2 引用特定对象的实例方法
```java
public interface Mytest1 {
    int test(String s);
}

//  Mytest1 mt2 = a -> "This is special".indexOf(a);  
    Mytest1 mt2 = "This is special"::indexOf; // 特定对象："This is .."的实例方法indexOf(String)
    int intb = mt2.test("is");
    System.out.println(intb);
```

### 15.3 引用类对象的实例方法
```java
public interface Mytest {
    String test(String a,int b, int c);
}

public class MytestDemo {
    public static void main(String[] args) {
//        Mytest mt = (a,b,c) -> a.substring(b,c);  lambda实现方式
        Mytest mt = String::substring;  // 先创建对象，引用该对象的实例方法

        String str = mt.test("abcdefg", 2,4);
        System.out.println(str);
    }
}

```


### 15.4 引用构造器
```java
public class Student {
    private int age;
    private String name;

    public Student(String name, int age) {
        this.age = age;
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public String getName() {
        return name;
    }
}

public interface StudentBuilder {
    Student build(String name, int age);
}


//        StudentBuilder sb = (a,b) -> new Student(a,b);
        StudentBuilder sb = Student::new;  
        Student sd = sb.build("haha", 10);
        System.out.println(sd.getName()+":"+sd.getAge());

```


## 16 接口组成成员

- 常量：`public static final`
- 抽象方法:`public abstract`,通常直接写`void method();`默认省略了`public abstract`
- 默认方法（java 8）：`public default`，用于在已经写好的程序中如果增加接口的一个方法，就要在实现接口的所有类中重写添加该方法。添加默认方法可以直接在接口中实现该方法体，且所有实现该接口的类中无需做任何改变。
- 静态方法（Java 8）：`public static`，在实现类中调用静态方法，需要通过接口调用，避免继承多接口时有重复的方法名。
- 私有方法（Java 9）：`private `，仅供内部使用，当内部的多个方法有重复的实现时，可以抽象出一个私有方法，供内部使用，但是不想让外部调用。
    - 默认方法可以调用私有的静态方法和非静态方法
    - 静态方法只能调用私有的静态方法

# 17 Stream流
使用Stream流对集合数据操作，先将集合数据转化为流数据，然后对流数据操作，最后也可以转化为集合数据

## 17.1 生成Stream流
```java
    //Collection体系的集合可以使用默认方法stream()生成流
    List<String> list = new ArrayList<String>();
    Stream<String> listStream = list.stream();
    Set<String> set = new HashSet<String>();
    Stream<String> setStream = set.stream();
    
    //Map体系的集合间接的生成流
    Map<String,Integer> map = new HashMap<String, Integer>();
    Stream<String> keyStream = map.keySet().stream();
    Stream<Integer> valueStream = map.values().stream();
    Stream<Map.Entry<String, Integer>> entryStream = map.entrySet().stream();
    
    //数组可以通过Stream接口的静态方法of(T... values)生成流
    String[] strArray = {"hello","world","java"};
    Stream<String> strArrayStream = Stream.of(strArray);
    Stream<String> strArrayStream2 = Stream.of("hello", "world", "java");
    Stream<Integer> intStream = Stream.of(10, 20, 30);
```

## 17.2 Stream流操作
| 方法名                                   | 说明                                                       |
| ---------------------------------------- | ---------------------------------------------------------- |
| void forEach(Consumer action)            | 对此流的每个元素执行操作                                   |
| long count()                             | 返回此流中的元素数                                         |
| Stream filter(Predicate predicate)       | 用于对流中的数据进行过滤                                   |
| Stream limit(long maxSize)               | 返回此流中的元素组成的流，截取前指定参数个数的数据         |
| Stream skip(long n)                      | 跳过指定参数个数的数据，返回由该流的剩余元素组成的流       |
| static Stream concat(Stream a, Stream b) | 合并a和b两个流为一个流                                     |
| Stream distinct()                        | 返回由该流的不同元素（根据Object.equals(Object) ）组成的流 |
| Stream sorted()                          | 返回由此流的元素组成的流，根据自然顺序排序                 |
| Stream sorted(Comparator comparator)     | 返回由该流的元素组成的流，根据提供的Comparator进行排序     |
| Stream map(Function mapper)              | 返回由给定函数应用于此流的元素的结果组成的流               |
| IntStream mapToInt(ToIntFunction mapper) | 返回一个IntStream其中包含将给定函数应用于此流的元素的结果  |

```java
        ArrayList<String> str = new ArrayList<String>(Arrays.asList("haha", "hello","hhhh","ahang"));
        
        str.stream().filter( s -> s.startsWith("h")).forEach(System.out::println);  
        // filter过滤后只有以h开头的字符串，forEach()对流中每个元素操作，可能会无序
        str.stream().filter( s -> s.startsWith("h") ).filter(s -> s.length() == 4).forEach(s -> System.out.println(s));
        
        str.stream().skip(1).limit(2).forEach(System.out::println);
        // skip跳过前几个， limit只保存前几个
        
        Stream<String> s1 = str.stream().skip(1);
        Stream<String> s2 = str.stream().limit(3);
        Stream.concat(s1, s2).forEach(System.out::println);  // concat合并两个流中元素，重复也存在
        Stream.concat(s1,s2).distinct().forEach(System.out::println);  // distinct去除重复的元素 

        str.stream().sorted().forEach(System.out::println); // 按照字符默认排序
        str.stream().sorted( (a,b) -> {           // 实现Comparetor接口
            int num1 = a.length() - b.length();
            int num2 = num1 == 0? a.compareTo(b) : num1;
            return num2;
        } ).forEach(System.out::println);
        
        ArrayList<String> list = new ArrayList<String> (Arrays.asList("10", "20", "40","9"));
        list.stream().map(s -> Integer.parseInt(s)).forEach(s -> System.out.println(s));
        list.stream().map(Integer::parseInt).forEach(System.out::println);
        int sum = list.stream().mapToInt(Integer::parseInt).sum();  
        // mapToInt()可以转化stream为IntStream，会有数字的相关方法
```

## 17.3 Stream流转化为集合
```java
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



# 18 源码解析

## 18.1 解析Arrays.sort() 

在对基本数据类型的数组排序时，Arrays.sort() 函数通过调用 DualPivotQuicksort.sort() 完成排序;

1. 当数组长度 >= 286，并且不存在较多连续相等元素，并且「高度结构化」时，采用类似 TimSort 的算法进行排序; 
   TimSort排序：先将数组划分多个子序有序数组，然后将数组归并排序，如

   ```
   [1,5,2,3] ==> [1,5] [2,3]
   [3,4,5,2] ==> [3,4,5] [2]
   [3,2,1,4,5] ==> [1,2,3] [4,5]  子序有序若逆序则翻转
   ```

2. 当数组长度 <= 47 时，采用插入排序或双插入排序;

3. 否则采用双轴快排进行排序。



# 19 比较器

## Comparable与Comparator

Comparable 简介
Comparable 是排序接口。
若一个类实现了Comparable接口，就意味着“该类支持排序”。此外，“实现Comparable接口的类的对象”可以用作“有序映射(如TreeMap)”中的键或“有序集合(TreeSet)”中的元素，而不需要指定比较器。
接口中通过x.compareTo(y)来比较x和y的大小。若返回负数，意味着x比y小；返回零，意味着x等于y；返回正数，意味着x大于y。

Comparator 简介
Comparator 是比较器接口。我们若需要控制某个类的次序，而该类本身不支持排序(即没有实现Comparable接口)；那么，我们可以建立一个“该类的比较器”来进行排序。这个“比较器”只需要实现Comparator接口即可。也就是说，我们可以通过“实现Comparator类来新建一个比较器”，然后通过该比较器对类进行排序。

int compare(T o1, T o2)和上面的x.compareTo(y)类似，定义排序规则后返回正数，零和负数分别代表大于，等于和小于。

注意点：

Comparator为功能接口可以使用lambda表达式

但是其中的compare方法不接受基本类型如int，需要引用类型如Integer；

comparingInt方法也只接受Integer类型，不接受int类型数组

比如：

```java
    int[] nums = new int[]{4,5,1,6,7,3,2};
	Arrays.sort(nums);  // 不能使用Comparator，因为只支持引用类型

    Integer[] numsInteger = new Integer[]{3,2,5,1};
    Arrays.sort(numsInteger, Comparator.comparingInt(o->o));

	int[][] numsArray = new int[3][2];
    Arrays.sort(numsArray, Comparator.comparing(a->a[0]-a[1]));
```





两者的联系
Comparable相当于“内部比较器”，而Comparator相当于“外部比较器”。

原文链接：https://blog.csdn.net/u010859650/article/details/85009595



## Collection与Collections的区别

**Collection**是[集合](https://so.csdn.net/so/search?q=集合&spm=1001.2101.3001.7020)类的上级**接口**，继承与他有关的接口主要有List和Set
**Collections**是针对集合类的一个**帮助类**，他提供一系列静态方法实现对各种集合的搜索、排序、线程安全等操作



# 泛型

E - Element (在集合中使用，因为集合中存放的是元素)

T - Type(Java 类型)

K - Key(键)

V - Value(值)

N - Number(数值类型)

？ -  表示不确定的java类型

## `<T>`和`<?>`

`“<T>"和"<?>"`，首先要区分开两种不同的场景：

1. 类型参数`“<T>”`主要用于声明泛型类或泛型方法。
```java
   class People<T>{
   public void show(T a) {
       
      }
   }
```

2. 无界通配符`“<?>”`主要用于使用泛型类或泛型方法
   `SuperClass<?> sup = new SuperClass<String>("lisi");`

参考：https://www.cnblogs.com/jpfss/p/9929045.html

https://www.zhihu.com/question/31429113



## `<? extends T>和<? super T>`
`ArrayList<? extends E> al = new ArrayList<? extends E>();`
        泛型的限定：
         ? extends E:接收E类型或者E的子类型。
         ？super E:接收E类型或者E的父类型。

参考：https://www.zhihu.com/question/20400700/answer/117464182

`<? extends T>` 上界通配符，适用范围为Fruit的子类

经常读取采用该方式

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/cdec0a066693684036d4bcaab4fdc1e3_1440w.jpg)



`<? super T>`下界通配符，适用于Fruit的父类、基类

经常插入数据采用该方式

![]( https://ahang.oss-cn-guangzhou.aliyuncs.com/img/0800ab14b2177e31ee3b9f6d477918fa_1440w.jpg)





作者：Sheldon Zhao
链接：https://zhuanlan.zhihu.com/p/110152973
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



# 反射

## 什么是反射？

反射是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为 Java 语言的反射机制。



## 反射的优点

- 反射机制极大的提高了程序的灵活性和扩展性，降低模块的耦合性，提高自身的适应能力。
- 通过反射机制可以让程序创建和控制任何类的对象，无需提前硬编码目标类。
- 反射机制是构建框架技术的基础所在，使用反射可以避免将代码写死在框架中。

正是反射有以上的特征，所以它能动态编译和创建对象，极大的激发了编程语言的灵活性，强化了多态的特性，进一步提升了面向对象编程的抽象能力。



## 反射的缺点

- 性能问题。Java反射机制中包含了一些动态类型，所以Java虚拟机不能够对这些动态代码进行优化。因此，反射操作的效率要比正常操作效率低很多。我们应该避免在对性能要求很高的程序或经常被执行的代码中使用反射。而且，如何使用反射决定了性能的高低。如果它作为程序中较少运行的部分，性能将不会成为一个问题。
- 安全限制。使用反射通常需要程序的运行没有安全方面的限制。如果一个程序对安全性提出要求，则最好不要使用反射。
- 程序健壮性。反射允许代码执行一些通常不被允许的操作，所以使用反射有可能会导致意想不到的后果。反射代码破坏了Java程序结构的抽象性，所以当程序运行的平台发生变化的时候，由于抽象的逻辑结构不能被识别，代码产生的效果与之前会产生差异。



# 静态变量和静态代码块的执行顺序 

执行顺序：

- 静态变量是全局变量
- 静态变量及代码块在类加载时被执行，只会被执行一次
- 静态变量和静态代码块按照声明的顺序依次执行
- 静态方法调用时才执行

1、类内容（静态变量、静态初始化块） => 实例内容（变量、初始化块、构造器）

2、父类的（静态变量、静态初始化块）=> 子类的（静态变量、静态初始化块）=> 父类的（变量、初始化块、构造器）=> 子类的（变量、初始化块、构造器）

```java
public class test {                         //1.第一步，准备加载类

    public static void main(String[] args) {
        new test();                         //4.第四步，new一个类，但在new之前要处理匿名代码块        
    }

    static int num = 4;                    //2.第二步，静态变量和静态代码块的加载顺序由编写先后决定 

    {
        num += 3;
        System.out.println("b");           //5.第五步，按照顺序加载匿名代码块，代码块中有打印
    }

    int a = 5;                             //6.第六步，按照顺序加载变量

    { // 成员变量第三个
        System.out.println("c");           //7.第七步，按照顺序打印c
    }

    test() { // 类的构造函数，第四个加载
        System.out.println("d");           //8.第八步，最后加载构造函数，完成对象的建立
    }

    static {                              // 3.第三步，静态块，然后执行静态代码块，因为有输出，故打印a
        System.out.println("a");
    }

    static void run()                    // 静态方法，调用的时候才加载// 注意看，e没有加载
    {
        System.out.println("e");
    }
}
```



静态变量及代码块在类加载时被执行，只会被执行一次

```java
public class StaticDemo5 {
    static int i=1;
    static{                            
        System.out.println("a");        //第1步。a。且只执行一次
        i++;                            //i=i+1，结果2
    }    
    public StaticDemo5(){
        System.out.println("c");        //第2步。
        i++;                            //i=i+1，结果3
    }
    
    public static void main(String[] args) {
        StaticDemo5 t1=new StaticDemo5();
        System.out.println(t1.i);        //第3步。3
        
        StaticDemo5 t2=new StaticDemo5();    //第4步。c。第二次创建实例。static静态代码块不执行。
        System.out.println(t2.i);        //第5步。又执行了一次StaticDemo5()构造函数。4
    }
}

输出：
a
c
3
c
4

创建第二个对象StaticDemo5 t2=new StaticDemo5()时，其实i已经先于对象存在于静态区域

```


## 枚举
https://blog.csdn.net/qq_27093465/article/details/52180865










