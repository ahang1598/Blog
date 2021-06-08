[TOC]


# 1. 方法

## 1.1 注意事项
- 方法不能嵌套定义
- 对于void类型方法，return可以不写 或着 只写一个return

## 1.2 通用格式
```java
public static 返回值类型 方法名():{
    方法体;
    return 返回值;
}
```

## 1.3 方法重载
- 多个方法在**同一个类**中
- 多个方法具有**相同的方法名**
- 多个方法的**参数不相同**，类型不同或者数量不同


- ==注意与返回值无关==
- 

# 2. 内部类
- 内部类方法可以访问该类定义所在的作用域中的数据 ， 包括私有的数据
- 内部类可以对同一个包中的其他类隐藏起来
- 当想要定义一个回调函数且不想编写大量代码时 ， 使用匿名（ anonymous ) 内部类比较便捷

## 2.0 权限修饰符
| 修饰符    | 同一个类中 | 同一个包中子类无关类 | 不同包的子类 | 不同包的无关类 |
| --------- | ---------- | -------------------- | ------------ | -------------- |
| private   | √          |                      |              |                |
| default   | √          | √                    |              |                |
| protected | √          | √                    | √            |                |
| public    | √          | √                    | √            | √              |

## 2.1 非静态内部类
==非静态内部类中不允许存在静态成员==

```java
public class OutClass {
    String vars = "外部类变量";  
    class InClass{
        String vars = "内部类变量";
        void method(){
            String vars = "局部变量";
            System.out.println(vars);  // 局部变量
            System.out.println(this.vars);  // 内部类变量
            System.out.println(MytestDemo.this.vars); // 外部类变量
        }
    }
    
    // 调用内部类的方法，不能在静态main中直接调用非静态内部类
    public void test(){
        InClass ic = new InClass();  // 外部类调用内部类的变量，要先创建内部类对象
        System.out.println("外部类调用内部类的变量："+ ic.vars); // 外部类调用内部类的变量：内部类变量
        ic.method();
    }
    
    public static void main(String[] args) {    
    //    new OutClass().new InClass().method(); // 先创建外部类再创建内部类也可以使用
        new OutClass().test();  // 非静态内部类调用
    }
}
```

## 2.2 静态内部类
> static关键字的作用是把类的成员变成类相关，而不是实例相关，即static修饰的成员属于整个类，而不属于单个对象。外部类的上一级程序单元是包，所以不可使用static修饰；而内部类的上一级程序单元是外部类，使用static修饰可以将内部类变成外部类相关，而不是外部类实例相关。因此static关键字不可修饰外部类，但可修饰内部类

- 静态内部类可以包含静态成员，也可以包含非静态成员
- 静态成员不能访问非静态成员
- 非静态成员不能访问外部非静态成员

```java
public class OutClass{
    private static String str = "静态外部变量";
    private String str1 = "非静态外部变量";  // 静态内部类无法访问
    
    static class StaticInClass{
        static String istr = "静态内部变量";
        String istr1 = "非静态内部变量";
        void method(){  // 非静态成员可以访问：（除外部非静态变量）
            System.out.println(istr); // 静态内部变量
            System.out.println(istr1); // 非静态内部变量
            System.out.println(str);  // 静态外部变量
//            System.out.println(str1);
        }
         
        static void staticMethod(){  // 静态成员可以访问：静态变量
            System.out.println(istr);     // 内部静态变量
//            System.out.println(istr1);
            System.out.println(str);     // 外部静态变量
//            System.out.println(str1);
        }
    }
    
    public static void main(String[] args) {
        new StaticInClass().method();    // 直接创建静态内部类对象，因为static将内部类变成了外部类相关
        new StaticInClass().staticMethod();  // 使用静态内部类的静态方法
        
        System.out.println(StaticInClass.istr); // 外部类调用静态内部类的静态内部成员
//        System.out.println(StaticInClass.istr1);  不能直接使用非静态内部变量
        System.out.println(new StaticInClass().istr1); // 可以通过创建对象调用非静态内部类
    }
}
```

## 2.3 局部内部类
定义在方法内的类

## 2.4 匿名内部类

```java
new 接口() | 父类构造器(){
    类体
}
```

```java
// 存在一个没有实现的接口
public interface Product {
    double getPrice();
    String getName();
}

// 正常需要通过创建一个类实现该接口的功能
class ProductInfo implements Product{
    public double getPrice(){ return 100.001; }
    public String getName(){ return "Peter"; }
}

// 但是通过匿名内部类则省略了实现该接口的类
public class AnoymousTest {
    public void test(Product p){
        System.out.println(p.getName() + " spend "+p.getPrice());
    }

    public static void main(String[] args) {
        AnoymousTest at = new AnoymousTest();
        
        // at.test( new ProductInfo() ); 内部类更改了这一步
        at.test(new Product() {
            @Override
            public double getPrice() {
                return 10.001;
            }

            @Override
            public String getName() {
                return "peter";
            }
        });
    }
}

```

```java
// 存在一个没有实现的抽象类
abstract class Product {
    public abstract double getPrice();  // 需要重写抽象方法
    private String name;
    public Product(){}
    public Product(String name){this.name = name;}  // 构造器

    public String getName() { return name; }
}

public class AnoymousTest {
    public void test(Product p){
        System.out.println(p.getName() + " spend "+p.getPrice());
    }

    public static void main(String[] args) {
        int localVar = 10;   // 局部变量
        AnoymousTest at = new AnoymousTest();
        at.test(new Product("Peter") {  // 通过带参构造器创建
            {
                System.out.println(age); 
                // 当使用了局部变量后，相当于系统自动改为final int localVar = 10;
                // age = 20; 报错，提示final变量不可变
            }
            @Override
            public double getPrice() {  // 重写方法
                return 100.1;
            }
        });
    }
}
```

匿名内部类可以访问局部变量，当访问时相当于自动加了`final`修饰，此时不可以对该变量进行任何修改。



# 3. 多态

## 3.1 向上向下转型

定义一个父类`Animals`包含`eat()`方法、`age`初始值为0

然后再定义两个子类：`Dog`和`Cat`,都重写了`eat()`方法和`age`，然后再分别有自己的特有方法狗咆叫`bark()`，猫睡觉`sleep()`

向上转型：`Animals a = new Cat()`

向下转型：`Cat cat = (Cat) a`,必须要先经过向上转型的变量--再向下转型，此时`cat`就拥有子类自己定义的特有方法`cat.sleep()`

`instanceof`用法：`a instanceof Cat` 这里`a`必须是经过向上转型后的父类对象，而`Cat`是继承父类的子类

```java
// 父类
public class Animals {
    public int age = 0;

    public void eat(){
        System.out.println("Eat...");
    }
}

// 子类 继承 父类 Animals
public class Dog extends Animals {
    public int age = 10;

    @Override
    public void eat(){
        System.out.println("eat bone...");
    }

    public void bark(){
        System.out.println("dog bark...");
    }
}

public class Cat extends Animals{
    public int age = 2;
    @Override
    public void eat(){
        System.out.println("eat fish...");
    }

    public void sleep(){
        System.out.println("cat sleep...");
    }
}

```

有一个宠物店`AnimalsShop`将宠物放入后，统计个数，以及可以取出该宠物
```java
public class AnimalsShop {
    // 注意这里用的Animals泛型，加入宠物时，不关心宠物的具体类型
    private List<Animals> list = new ArrayList<Animals>();
    // 当加入一个宠物时，相当于进行了向上转型
    public void add(Animals a){
        list.add(a);
    }

    public int getSize(){
        return list.size();
    }
    // 取出的宠物也是经过向上转型的，此时类型为Animals
    public Animals getAnimal(int position){
        return list.get(position);
    }
}
```

```java
public static final int DOG = 0;
public static final int CAT = 1;
    @Test
    public void test4(){
        // 通过泛型进行向上转型将不同类型的动物加入集合，此时不需要动物的不同特性，只需要知道个数即可
        AnimalsShop as = new AnimalsShop();
        as.add(new Dog());
        as.add(new Cat());

        // 获取动物个数
        System.out.println(as.getSize());

        // 查看所有的宠物
        Dog dog = (Dog) as.getAnimal(DOG); // 直接向下转型，并不安全，因为可以编译通过但运行出错
        dog.eat();
        dog.bark();

        Animals a = as.getAnimal(CAT);  // 获取已经向上转型Cat
        System.out.println(a instanceof Dog); // false，向上转型的Cat和Dog不构成子类关系
        // (（已经经过向上转型的）父类对象 instanceof 子类 ) 来判断是否可以向下转型更安全，防止ClassCastException转换异常
        if(a instanceof Cat){
            Cat cat = (Cat) as.getAnimal(CAT);
            cat.eat();
            cat.sleep();
        }

    }
```

> 向下转型的最大好处是：java的泛型编程，用处很大，Java的集合类都是这样的。
> 参考博客
>
> [Java向下转型的意义](https://blog.csdn.net/xyh269/article/details/52231944)
>
> [Java对象类型向上转型](https://blog.csdn.net/yuncaidaishu/article/details/88690799)

## 3.2 多态的特点
多态的前提：
- 存在继承或实现接口`extend`或`implement`
- 重写方法`@Override`
- 父类引用指向子类对象`Animals a = new Cat()`

多态的特点：
- 相同变量看父类`a.age = 0`
- 重写方法看子类`eat()看子类`
- 子类特有方法不可用`a.sleep()不可用`



```java
    public static void animalEat(Animals a){
        a.eat();
    }

    @Test
    public void test3(){
        Animals a1 = new Dog();
        System.out.println(a1.age); // 1. 变量看父类
        a1.eat(); // 2. 方法看子类
//        a1.bark();  3. 子类特有不可用

        Animals a2 = new Cat();
        a2.eat();

        // 当向上转型后，通过一个通用方法调用不同的子类，即可实现子类的方法
        animalEat(new Dog());
        animalEat(new Cat());
    }
```