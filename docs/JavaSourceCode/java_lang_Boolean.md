# java.lang.Boolean 阅读总结

## 概要

**Boolean** 是基础数据类型**boolean**的包装类，有**true**和**false**两种情况。它实现了Serializable以及Comparable<>接口。

它有两个静态的自身实例分别为**TRUE**以及**FALSE**，它们用于包装私有属性的**true**和**false**，如果不需要一个新的实例使用它们你将会获得时间和空间上的收益。

还有一个呢就是TYPE了，它是用于包装class里的私有属性boolean的。

它用一个属性 **value**就是存储实例的值。如下:

```java

	/**
	* 以下三个用于包装class里的boolean数据类型
	*/    
	public static final Boolean TRUE = new Boolean(true);

    public static final Boolean FALSE = new Boolean(false);

	public static final Class<Boolean> TYPE = (Class<Boolean>) Class.getPrimitiveClass("boolean");

	/**
	* 存储对象值，是true还是false
	*/
	private final boolean value;

	/**
	* 序列化id
	*/
	private static final long serialVersionUID = -3665804199014368530L;
```

## 方法

### 构造方法

```java
	/**
     * 分配一个Boolean给boolean数据类型的value值。
     */
    public Boolean(boolean value) {
        this.value = value;
    }

    /**
     * 如果String对象s的值不为空并且s.equalsIgnoreCase("true")为真
     * 分配一个值为true的Boolean对象。
     * 否则返回值为false的Boolean对象。
     */
    public Boolean(String s) {
        this(parseBoolean(s));
    }
	
	/**
     * 如果String对象s的值不为空并且s.equalsIgnoreCase("true")为真返回true。
     * 否则返回false。
     */
	public static boolean parseBoolean(String s) {
        return ((s != null) && s.equalsIgnoreCase("true"));
    }
```

### 其他方法

#### boolean booleanValue()

返回该Boolean对象的值。

#### static Boolean valueOf(boolean b)

如果boolean为true，返回Boolean.TRUE，为false返回Boolean.FALSE。并不新建Boolean实例。

如果不需要一个新的Boolean实例，那么通常应该优先使用该方法，而不是构造函数 Boolean (boolean)，因为该方法很可能产生更好的空间和时间性能。

#### static Boolean valueOf(String s)

如果String对象s的值不为空并且s.equalsIgnoreCase("true")为真，返回Boolean.TRUE，否则返回Boolean.FALSE。

#### static String toString(boolean b)

b为true返回"true"，否则返回"false"。

#### String toString()

b为true返回"true"，否则返回"false"。

#### int hashCode()

返回hash值，调用了static int hashCode 方法，如下。

```java 
    /**
     * 我们可以很明显看出来，Boolean的hashCode值是定死的，不是1231(true)就是1237(false)。
     */
	public static int hashCode(boolean value) {
        return value ? 1231 : 1237;
    }
```

#### boolean Equals(Object obj)

看看源码就知道它们比较的是值了，如果同为true或者同为false，返回true，否则返回false。

```java
    public boolean equals(Object obj) {
        if (obj instanceof Boolean) {
            return value == ((Boolean)obj).booleanValue();
        }
        return false;
    }
```

#### static boolean getBoolean(String name)

如果系统变量里有一个名为name的变量并且它与"true"是相等的，返回true。否则返回false。

```java
    public static boolean getBoolean(String name) {
        boolean result = false;
        try {
            result = parseBoolean(System.getProperty(name));
        } catch (IllegalArgumentException | NullPointerException e) {
        }
        return result;
    }
```

#### int compareTo(Boolean b)

b不为空的情况下：

- 两个相同返回0；
- b为false返回1；
- b为true返回-1；

```java

    public int compareTo(Boolean b) {
        return compare(this.value, b.value);
    }

    /**
     * @return the value {@code 0} if {@code x == y};
     *         a value less than {@code 0} if {@code !x && y}; and
     *         a value greater than {@code 0} if {@code x && !y}
     * @since 1.7
     */
    public static int compare(boolean x, boolean y) {
        return (x == y) ? 0 : (x ? 1 : -1);
    }
```

#### 浴后妃

```java
    /**
     * a 与 b 全为真
     */
    public static boolean logicalAnd(boolean a, boolean b) {
        return a && b;
    }

    /**
     * a 或者 b中有一个为真
     */
    public static boolean logicalOr(boolean a, boolean b) {
        return a || b;
    }

    /**
     * a b 不同返回真，相同返回假
     */
    public static boolean logicalXor(boolean a, boolean b) {
        return a ^ b;
    }
```

## 扩展

### Boolean 到底占几个字节

大家可能都知道，java与其他语言不一样，基础数据类型占几个字节是规定好了的比如int 4个字节，long 8个字节，但是Boolean到底占几个字节？你真的确定只占一个嘛？

Java虚拟机规范一书提到 :

- 在Java虚拟机中没有任何供 boolean值专用的字节码指令,Java语言表达式所操作的
  boolean值,在编译之后都使用**Java虚拟机中的int数据类型**来代替。(4字节)
- Java虚拟机直接支持 boolean类型的数组,虚拟机的 navarra指令参见第6章的newarray小节可以创建这种数组。**boolean类型数组的访问与修改共用byte类型数组的baload和 bastore指令**。(1字节)

> 那虚拟机为什么要用int来代替boolean呢？为什么不用byte或short，这样不是更节省内存空间吗。经过查阅资料发现，使用int的原因是，对于当下32位的处理器（CPU）来说，一次处理数据是32位（这里不是指的是32/64位系统，而是指CPU硬件层面），32 位 CPU 使用 4 个字节是最为节省的，哪怕你是 1 个 bit 他也是占用 4 个字节。因为 CPU 寻址系统只能 32 位 32 位地寻址，具有高效存取的特点。

但是**具体的占用多少个字节要看JVM的实现，因为并没有明确的规定Boolean要占几个字节。**

#### 测试

我们可以自己写个小例子测试一下，如：

```java
package com.java.test.lang;

public class BooleanTest {

    void Int(){
        int i = 1;
    }

    void Boo(){
        boolean b = true;
    }

    void ByteArray(){
        byte[] bArr = new byte[10];
    }

    void BooArray(){
        boolean[] bArr = new boolean[10];
    }
}
```

javac 编译然后 javap -v 查看字节码，如下(为方便阅读，删掉了其他如常量池以及不必要的显示)：

```java
public class com.java.test.lang.BooleanTest
  minor version: 0
  major version: 56
{
  void Int();
    descriptor: ()V
    flags: (0x0000)
    Code:
      stack=1, locals=2, args_size=1
         0: iconst_1
         1: istore_1
         2: return
      LineNumberTable:
        line 6: 0
        line 7: 2

  void Boo();
    descriptor: ()V
    flags: (0x0000)
    Code:
      stack=1, locals=2, args_size=1
         0: iconst_1
         1: istore_1
         2: return
      LineNumberTable:
        line 10: 0
        line 11: 2

  void ByteArray();
    descriptor: ()V
    flags: (0x0000)
    Code:
      stack=1, locals=2, args_size=1
         0: bipush        10
         2: newarray       byte
         4: astore_1
         5: return
      LineNumberTable:
        line 14: 0
        line 15: 5

  void BooArray();
    descriptor: ()V
    flags: (0x0000)
    Code:
      stack=1, locals=2, args_size=1
         0: bipush        10
         2: newarray       boolean
         4: astore_1
         5: return
      LineNumberTable:
        line 18: 0
        line 19: 5
}
```

可以看到boolean的操作用的是int或者是byte的字节码命令。

参考链接:

[Java中boolean类型到底占用多少个字节？](https://blog.csdn.net/YuanMxy/article/details/74170745)

[腾讯面试官问我Java中boolean类型占用多少个字节？我说一个，面试官让我回家等通知](https://juejin.im/post/5dec63fe6fb9a0166138f44d)