[TOC]



## 1. 类型转换

**自动转换** `由小到大` 

`byte、short、char‐‐>int‐‐>long‐‐>float‐‐>double`

```java
double num1 = 100F;  // 100.0
float num2 = 1000L;  //1000.0
long num3 = 10;    // 10
```

**强制转换**
```
byte a = 10;
byte b = 10;
byte c = (byte)(a + b); // a+b为int型，需要强制转化为byte型
```
**注意事项**

1. 精度损失
2. short、byte、char 可以加减，自动转换为int型然后再计算

## 2. Scanner
`import java.util.Scanner;`
### 2.1 三种输入数据流方式：
```java
// 用户输入
    Scanner sc = new Scanner(System.in);
    int i = sc.nextInt();

// 文件读取
      Scanner sc = new Scanner(new File("myNumbers"));
      while (sc.hasNextLong()) {
          long aLong = sc.nextLong();
      }

// 字符串读取
     String input = "1 fish 2 fish red fish blue fish";
     Scanner s = new Scanner(input).useDelimiter("\\s*fish\\s*");
     System.out.println(s.nextInt());
     System.out.println(s.nextInt());
     System.out.println(s.next());
     System.out.println(s.next());
     s.close();
```

### 2.2 常用方法
```java
sc.hasNextLine() // 如果存在下一行则返回 true
sc.nextInt()   // 读取Int类型数据
sc.nextLong()  // 读取Long类型数据, 不同类型都有Float,Byte,Short,Boolean
sc.nextLine() //读取下一行
```

## 3. 输出控制

必须通过`System.out.printf( "%f" )`进行格式控制

`%8.2f`表示符号长度为8，精确小数到2位；`3333.33333334`转换为` 3333.33`7位字符和1位空格最前面

| 标志 | 说明                                                     | 示例                         | 结果                 |
| ---- | -------------------------------------------------------- | ---------------------------- | -------------------- |
| +    | 为正数或者负数添加符号                                   | ("%+d",15)                   | `+15`                |
| −    | 左对齐                                                   | ("%-5d",15)                  | `|15  |`             |
| 0    | 数字前面补0                                              | ("%04d", 99)                 | `0099`               |
| 空格 | 在整数之前添加指定数量的空格                             | ("% 4d", 99)                 | `|   99|`            |
| ,    | 以“,”对数字分组                                          | ("%,f", 9999.99)             | `9,999.990000`       |
| (    | 使用括号包含负数                                         | ("%(f", -99.99)              | `(99.990000)`        |
| #    | 如果是浮点数则包含小数点，如果是16进制或8进制则添加0x或0 | `("%#x", 99)`，`("%#o", 99)` | `0x63，0143`         |
| <    | 格式化前一个转换符所描述的                               | `("%f和%<3.2f", 99.45)`      | `99.450000`和`99.45` |
| $    | 被格式化的参数索引                                       | `("%1$d,%2$s", 99,"abc")`    | `99,abc`             |


## 4. Switch
```java
switch(表达式){
    case 1:
    case 2:
        语句体1;
        break;
    case 3:
        语句体2;
        break;
    default:
        语句体3;
        break;
}


```

## 5. Random
`import java.util.Random`

常用方法：
```java
Random ra = new Random();

ra.setSeed(种子值)
ra.nextInt()
ra.nextInt(正数边界值)

```

- 生成某范围的随机数
```
    Random r = new Random();
    public int getRandom(int min, int max){
        int num = r.nextInt(max);
        num = num  % (max-min+1) + min;
        return num;
    }
```

## 6. 堆内存，方法栈

- 堆内存： `new出来的对象、数组都存储在堆内存`
- 方法栈： `方法运行时使用内存，比如main方法运行时，进入方法栈中执行`

### 6.1 多数组指向相同内存
![image](https://note.youdao.com/yws/public/resource/9a17bf7311b3424a6840aae93c5a8648/xmlnote/34E2CD9C801342A5A6FD51F6F2F18186/18956)

## 7. 数组

- 静态初始化
`int[] a = {1,2,3,4}`

- 动态初始化
`int[] a = new int[5]`
默认：
`int,long,byte,short`: 0

`float, double`: 0.0

`char`: `\u0000`

`boolean`: false

引用类型（数组、类、接口）：`null`


- 数组拷贝
```java
        int[] arr = new int[3]
        int[] arrcopy = Arrays.copyOf(arr, 2*arr.length); // 复制数组，而非引用
        int[] arrcopy1 = Arrays.copyofRange(arr, 1, arr.length-1) // 复制数组的一部分
        
        for(int i = 0; i< arrcopy.length; i++) arrcopy[i]+= 1; // 修改复制后数组
        for(int i = 0; i< arr.length; i++) arr[i]+= 2;  // 修改原数组
        
        System.out.println(Arrays.toString(arr));   // [2,2,2]
        System.out.println(Arrays.toString(arrcopy));  // [1,1,1,1,1,1] 两个数组不会互相影响，完全独立
```

- 数组排序
`Arrays.sort(arr)`

- 数组填充
`Arrays.fill(arr, 10);`

- 数组一致性对比
`Arrays.equals(arr1, arr2)`

- 二维数组
```java
        // 声明二维数组
        int[][] arr2 = {
                {1,2,3},
                {2,3,4},
                {3,4,5}
        };
        
        // 遍历二维数组输出的几种方式
        for(int i = 0; i < arr2.length; i++){
                System.out.println(Arrays.toString(arr2[i]));
        }
[1, 2, 3]
[2, 3, 4]
[3, 4, 5]        
        // 标准对二维数组操作
        for(int i = 0; i < arr2.length; i++){
            for( int j = 0; j < arr2[i].length; j++){
                System.out.print(arr2[i][j]);
            }
            System.out.println();
        }
123
234
345
        // 快速遍历读取
        for(int[] row: arr2)
            for(int e: row)
                System.out.print(e);
123234345
        // 转化为数组读取

        System.out.println(Arrays.deepToString(arr2));
[[1, 2, 3], [2, 3, 4], [3, 4, 5]]

```

- 二维数组的行互换
```java
        int[] tmp = arr2[1];
        arr2[1] = arr2[2];
        arr2[2] = tmp;
```

- 不规则数组
```java
        // 不规则数组声明
        int[][] arr3 = new int[5][];
        for(int i = 0; i < 5; i++){
            arr3[i] = new int[i+1];
        }
        
        // 赋值
        for(int i = 0; i < arr3.length; i++){
            for(int j = 0; j < arr3[i].length; j++){
                if(j == 0 | i==j){ arr3[i][j] = 1;}
                else arr3[i][j] = arr3[i-1][j] + arr3[i-1][j-1];
            }
        }
        
        // 遍历
        for(int i = 0; i < arr3.length; i++){
            System.out.println(Arrays.toString(arr3[i]));
        }
[1]
[1, 1]
[1, 2, 1]
[1, 3, 3, 1]
[1, 4, 6, 4, 1]
```




## 7. for循环

- for each不需要下标的遍历数组
```java
    for(int e: arr) System.out.println(e);
        
    for(int i = 0; i < arr.length; ++i){
        System.out.println(arr[i]);
    }
```

或者直接通过Arrays.toString(arr)转换为字符串输出：`System.out.println(Arrays.toString(arr));` --》 `[0, 0, 0]`


## 8. 数据类型转换



### 8.2 int,long,float,double,boolean,char,char[] --> String
`String s = String.valueOf(10)`
`String s = String.valueOf(10.2)`
`String s = String.valueOf(true)`

### 8. Integer,Long,Float,Double,Boolean,Char,Char[] --> String
`Double d = 10.2;  String ds = d.toString();`

`Integer i = 10;  String is = i.toString();`

`Boolean b = true;  String bs = b.toString();`

需要注意的是：这里`int,double...`基本类型不可以使用，只有`Integer,Double`包装类才可以使用`toString()`方法

### 8.1 String --> int
`int i = Integer.parseInt("10")`

### 8.2 String --> Double
`Double d = Double.valueOf("10.2")`

### String --> char[]
`String str="123456";  char[] c = str.toCharArray() ;`

###  Double --> int
`Double d = 10.6; int di = d.intValue();`得到整数10，舍去小数位

### Date <--> String
```java
java.sql.Date day = TypeChange.stringToDate("2003-11-3");
String strday = TypeChange.dateToString(day);
```

### String --> Integer, Long, Float, Double, Short, Byte
```java
string->byte 
Byte static byte parseByte(String s)  

string->Short 
Short static Short parseShort(String s) 

String->Integer 
Integer static int parseInt(String s) 

String->Long 
Long static long parseLong(String s) 

String->Float 
Float static float parseFloat(String s) 

String->Double 
Double static double parseDouble(String s) 
```

## 9. 各个数据类型默认值
所有的包装类`Integer,Double...`默认值都为`null`

| 序号 | 数据类型       | 大小/位 | 封装类    | 默认值         | 可表示数据范围                           |
| ---- | -------------- | ------- | --------- | -------------- | ---------------------------------------- |
| 1    | byte(位)       | 8       | Byte      | 0              | -128~127                                 |
| 2    | short(短整数)  | 16      | Short     | 0              | -32768~32767                             |
| 3    | int(整数)      | 32      | Integer   | 0              | -2147483648~2147483647                   |
| 4    | long(长整数)   | 64      | Long      | 0L             | -9223372036854775808~9223372036854775807 |
| 5    | float(单精度)  | 32      | Float     | 0.0f           | 1.4E-45~3.4028235E38                     |
| 6    | double(双精度) | 64      | Double    | 0.0            | 4.9E-324~1.7976931348623157E308          |
| 7    | char(字符)     | 16      | Character | '/uoooo'(null) | 0~65535                                  |
| 8    | boolean        | 8       | Boolean   | flase          | true或false                              |

### 9.2 int和Integer的区别

1、`Integer`是`int`的包装类，`int`则是`java`的一种基本数据类型 
2、`Integer`变量必须实例化后才能使用，而`int`变量不需要 
3、`Integer`实际是对象的引用，当`new`一个`Integer`时，实际上是生成一个指针指向此对象；而int则是直接存储数据值 
4、`Integer`的默认值是`null`，`int`的默认值是`0`

**关于Integer和int的比较** [参考](https://www.cnblogs.com/guodongdidi/p/6953217.html)
1、由于Integer变量实际上是对一个Integer对象的引用，所以两个通过new生成的Integer变量永远是不相等的（因为new生成的是两个对象，其内存地址不同）。

```java
Integer i = new Integer(100);
Integer j = new Integer(100);
System.out.print(i == j); //false
```

2、Integer变量和int变量比较时，只要两个变量的值是向等的，则结果为true（因为包装类Integer和基本数据类型int比较时，java会自动拆包装为int，然后进行比较，实际上就变为两个int变量的比较）

```java
Integer i = new Integer(100);
int j = 100；
System.out.print(i == j); //true
```

3、非new生成的Integer变量和new Integer()生成的变量比较时，结果为false。（因为 ①当变量值在-128~127之间时，非new生成的Integer变量指向的是java常量池中的对象，而new Integer()生成的变量指向堆中新建的对象，两者在内存中的地址不同；②当变量值在-128~127之间时，非new生成Integer变量时，java API中最终会按照new Integer(i)进行处理（参考下面第4条），最终两个Interger的地址同样是不相同的）

```java
Integer i = new Integer(100);
Integer j = 100;
System.out.print(i == j); //false
```

4、对于两个非new生成的Integer对象，进行比较时，如果两个变量的值在区间-128到127之间，则比较结果为true，如果两个变量的值不在此区间，则比较结果为false

```java
Integer i = 100;
Integer j = 100;
System.out.print(i == j); //true
Integer i = 128;
Integer j = 128;
System.out.print(i == j); //false
```

对于第4条的原因： 
java在编译Integer i = 100 ;时，会翻译成为Integer i = Integer.valueOf(100)；，而java API中对Integer类型的valueOf的定义如下：

```java
public static Integer valueOf(int i){
    assert IntegerCache.high >= 127;
    if (i >= IntegerCache.low && i <= IntegerCache.high){
        return IntegerCache.cache[i + (-IntegerCache.low)];
    }
    return new Integer(i);
}
```

java对于-128到127之间的数，会进行缓存，Integer i = 127时，会将127进行缓存，下次再写Integer j = 127时，就会直接从缓存中取，就不会new了

