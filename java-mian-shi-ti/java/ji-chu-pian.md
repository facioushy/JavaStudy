# 基础篇

## 1. Java中操作字符串都有哪些类？它们之间的区别

有：String`、`StringBuffer`、`StringBuilder

String不可变，每次操作都会生成新的String对象，然后将指针指向新的对象，而StringBuffer和StringBuilder在原有对象的基础上进行操作。

String初始化两种方式的区别：

1.

```java
String str = "aa";
```

只会在字符串常量池中生成新的对象

2.

```java
String str = new String("aa");
```

会在字符串常量池和堆中都生成新的对象

而StringBuffer 和 StringBuilder 最大的区别在于，StringBuffer 是线程安全的，而 StringBuilder 是非线程安全的，但 StringBuilder 的性能却高于 StringBuffer，所以在单线程环境下推荐使用 StringBuilder，多线程环境下推荐使用 StringBuffer。

|  | String | StringBuffer | StringBuilder |
| :--- | :--- | :--- | :--- |
| 是否可变 | 不可变\(final修饰\) | 可变 | 可变 |
| 功能介绍 | 每次对String的操作都会在“常量池”中生成新的String对象 | 任何对它指向的字符串的操作都不会产生新的对象。每个StringBuffer对象都有一定的缓冲区容量，字符串大小没有超过容量时，不会分配新的容量，当字符串大小超过容量时，自动扩容 | 功能与StringBuffer相同，相比少了同步锁，执行速度更快 |
| 是否线程安全 | 是 | 是 | 否 |
| 使用场景 | 单次操作或循环外操作字符串 | 多线程操作字符串 | 单线程操作字符串 |

### 三者效率对比

> StringBulider &gt; StringBuffer &gt; String

对String进行改变时，我们需要在栈堆内存中开辟内存空间的，所以是最慢的

StringBuffer和StringBuilder主要操作是append和insert，append直接加在最后，insert在指定点加入。

String 类型和 StringBuffer、 StringBuild类型的主要性能区别其实在于 String 是不可变的对象（final）, 因此在每次对 String 类型进行改变的时候其实都等同于在堆中生成了一个新的 String 对象，然后将指针指向新的 String 对象，这样不仅效率低下，而且大量浪费有限的内存空间，所以经常改变内容的字符串最好不要用 String 。因为每次生成对象都会对系统性能产生影响，特别是当内存中的无引用对象过多了以后， JVM 的 GC 开始工作，那速度是一定会相当慢的。另外当GC清理速度跟不上new String的速度时，还会导致内存溢出Error，会直接kill掉主程序

### StringBuilder和StringBuffer线程安全主要差在哪？

StringBuffer和StringBuilder可以算是双胞胎了，这两者的方法没有很大区别。但在线程安全性方面，StringBuffer允许多线程进行字符操作。这是因为在源代码中StringBuffer的很多方法都被关键字synchronized 修饰了，而StringBuilder没有。

## 2. Error和Exception区别

Error 和 Exception 都是 Throwable 的子类，在Java中只有Throwable类型的实例才可以被抛出或者捕获，它是异常处理机制的基本类型。

1. Exception和Error体现了java平台设计者对不同异常情况的分类，Exception是程序正常运行中，可以预料的意外情况，可能并且应该被捕获，进行相应的处理。
2. Error是指正常情况下，不大可能出现的情况，绝大部分的Error都会导致程序处于非正常的、不可恢复的状态。既然是非正常情况，不便于也不需要捕获。常见的比如OutOfMemoryError之类都是Error的子类。
3. Exception又分为可检查\(checked\)异常和不可检查\(unchecked\)异常。可检查异常在源代码里必须显式的进行捕获处理，这是编译期检查的一部分。不可检查时异常是指运行时异常，像NullPointerException、ArrayIndexOutOfBoundsException之类，通常是可以编码避免的逻辑错误，具体根据需要来判断是否需要捕获，并不会在编译期强制要求。

## 3. ==和equals区别

==：判断地址是否相同，基本数据类型比较的是值，引用数据类型比较地址

equals：两种情况：

1. 类没有覆盖equals方法，则对比时等价于调用了Object类的equals\(\)方法，也就是通过"=="比较。

   ```java
   // Object类中的equals() 方法
   public boolean equals(Object obj) {
       return (this == obj);
   }
   ```

2. 类覆盖了equals\(\) 方法。一般，我们都会覆盖 equals\(\) 方法来两个对象的内容相等；若它们的内容相等，则返回 true \(即，认为这两个对象相等\)。

   ```java
   // String类中的equals() 方法，已覆盖，用于比较内容
   public boolean equals(Object anObject) {
       if (this == anObject) {
           return true;
       }
       if (anObject instanceof String) {
           String anotherString = (String)anObject;
           int n = value.length;
           if (n == anotherString.value.length) {
               char v1[] = value;
               char v2[] = anotherString.value;
               int i = 0;
               while (n-- != 0) {
                   if (v1[i] != v2[i])
                       return false;
                   i++;
               }
               return true;
           }
       }
       return false;
   }
   ```

### 如果不重写equals\(\)方法会怎样

举例:

```java
public static void main(String[] args) {
    // 字符串比较
    String a = "陈哈哈";
    String b = "陈哈哈";

    if (a == b) {// true  a==b
        System.out.println("a==b");
    }
    if (a.equals(b)) {// true  a.equals(b)
        System.out.println("a.equals(b)");
    }

    // StringBuffer 对象比较，由于StringBuffer没有重写Object的equal方法，因此结果出现错误
    StringBuffer c = new StringBuffer("陈哈哈");
    StringBuffer d = new StringBuffer("陈哈哈");

    if (c == d) {// false  c != d
        System.out.println("c == d");
    } else {
        System.out.println("c != d");
    }

    if (c.equals(d)) { // false 调用了Object类的equal方法
        System.out.println("StringBuffer equal true");
    }else {
        System.out.println("StringBuffer equal false");
    }
}
```

* `object的equals方法是比较的对象的内存地址，而String的equals方法比较的是对象的值。`
* 因为String中的equals方法是被重写过的，而StringBuilder没有重写equals方法，从而调用的是Object类的equals方法，也就相当于用了 `==`；

### 重写equals时，为什么要重写hashcode方法

在重写equals\(\)方法时，也有必要对hashCode\(\)方法进行重写，尤其是当我们自定义一个类，想把该类的实例存储在集合中时。 hashCode方法的常规约定为：值相同的对象必须有相同的hashCode，也就是equals\(\)结果为相同，那么hashcode也要相同，equals\(\)结果为不相同，那么hashcode也不相同； 当我们使用equals方法比较说明对象相同，但hashCode不同时，就会出现两个hashcode值，比如在HashMap中，就会认为这是两个对象，因此会出现矛盾，说明equals方法和hashCode方法应该成对出现，当我们对equals方法进行重写时，也要对hashCode方法进行重写。

## 为什么重写equals必须重写hashcode

如果只重写equals方法，但不重写hashcode，两个仅内存地址不同但变量完全相同的对象会分别存入不同的桶。

