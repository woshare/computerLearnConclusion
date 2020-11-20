# 谈谈String

## String对象堆栈布局结构 

![String对象堆栈布局结构](./res/string-intern-mem-struct.png "String对象堆栈布局结构")

![String对象堆栈布局结构](./res/string-mem-struct.png "String对象堆栈布局结构")

## String 类
```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
 /** The value is used for character storage. */
    private final char value[];
    private int hash;
    }
```

## String、StringBuffer和StringBuilder的区别
### 1，可变性
>1，为了线程安全和JVM缓存速度，String 对象是不可变的（被final修饰）。>2，StringBuilder 与 StringBuffer 都继承自 AbstractStringBuilder 类，在 AbstractStringBuilder 中也是使用字符数组保存字符串char[]value 但是没有用 final 关键字修饰，所以这两种对象都是可变的

### 2，线程安全：
>1，String 中的对象是不可变的，也就可以理解为常量，线程安全
>2，StringBuffer 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。
>3，StringBuilder 并没有对方法进行加同步锁，所以是非线程安全的  



### String +运算符 原理
>JVM隐式创建StringBuilder的方式在大部分情况下并不会造成效率的损失，不过在进行大量循环拼接字符串时则需要注意，以及3，StringBuilder不是线程安全的

```
public class Test {
    public static void main(String[] args) {
        int i = 10;
        String s = "abc";
        System.out.println(s + i);
    }
反编译后代码变为：
public class Test {
    public static void main(String args[]) {    //删除了默认构造函数和字节码
        byte byte0 = 10;      
        String s = "abc";      
        System.out.println((new StringBuilder()).append(s).append(byte0).toString());
    }
}
```

## final关键字
>1，final关键字主要用在三个地方：变量、方法、类
>2，final变量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。
>3，final修饰一个类时，表明这个类不能被继承。final类中的所有成员方法都会被隐式地指定为final方法。
>4，final方法的原因有两个。第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。在早期的Java实现版本中，会将final方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升（现在的Java版本已经不需要使用final方法进行这些优化了）。类中所有的private方法都隐式地指定为final。

### final修饰作用
>1，final的关键字提高了变量性能，JVM和java应用会缓存final变量；
>2，final变量可以在多线程环境下保持线程安全；
>3，使用final的关键字提高了方法性能，JVM会对方法变量类进行优化；