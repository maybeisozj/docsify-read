# java.lang.StringBuilder 阅读总结

## 概要

StringBuilder继承了AbstractStringBuilder抽象类，它与StringBuffer几乎是一样的，除了他们的安全性以及由此导致的速度问题(**StringBuffer是线程安全的，但是他的速度比StringBuilder慢了10~20%**)。StringBuffer以及StringBuilder是为了解决String一旦修改就要产生新对象的问题而产生的。

它有以下属性：

- char[] value //用于存储字符串，继承自AbstractStringBuilder
- int count// 存储字符串的的个数，继承自AbstractStringBuilder

**注意这里就没有toStringCache了。**因为没有必要欧。

几个参考网站:

[Why StringBuffer has a toStringCache while StringBuilder not?](https://stackoverrun.com/cn/q/12697236)

[老生常谈：StringBuilder 和 StringBuffer 的那些事儿](https://juejin.im/post/5df8f7eae51d45581a70a53e)

## 方法

### 构造方法

构造方法没有太大变化。

```java
	/**
    * 默认value容量为 16
    */
    public StringBuilder() {
        super(16);
    }

	/**
	* 指定value容量为capacity
	* capacity 为负数抛出 NegativeArraySizeException 异常
	*/
    public StringBuilder(int capacity) {
        super(capacity);
    }

	/**
	* 以String 进行初始化并增加16的容量
	*/
    public StringBuilder(String str) {
        super(str.length() + 16);
        append(str);
    }

	/**
	* 以CharSequence 进行初始化并增加16的容量
	*/
    public StringBuilder(CharSequence seq) {
        this(seq.length() + 16);
        append(seq);
    }
```

### 其他方法

细看源码，重写的方法体里调用的全是AbstractStringBuilder的方法，乌拉。

#### 序列化

[Java 序列化](https://juejin.im/post/5ce3cdc8e51d45777b1a3cdf)

```java
	/**
     * 序列化
     */
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        s.defaultWriteObject();
        s.writeInt(count);
        s.writeObject(value);
    }
	/**
     * 反序列化
     */
	private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();
        count = s.readInt();
        value = (char[]) s.readObject();
    }
```

## 扩展

### Buffer与Builder的联系与区别

#### 联系

1. 全部继承了AbstractStringBuilder抽象类，实现了CharSequence接口；
2. 全部用来做字符串的变更；
3. 初始容量都是16和扩容机制都是"旧容量*2+2"；

#### 区别

1. StringBuffer线程安全，StringBuilder非线程安全；
2. StringBuffer从JDK1.0就有了，StringBuilder是JDK5.0才出现；
3. StringBuffer比StringBuilder多了一个toStringCache字段，用来在toString方法中进行缓存，每次变更字符串的操作之前都先把toStringCache设置为null，若多次连续调用toString方法，可避免每次Arrays.copyOfRange(value, 0, count)操作，节省性能。
4. 由于StringBuilder没有考虑同步，在单线程情况下，StringBuilder的性能要优于StringBuffer；(**为什么呢，可以看synchronized关键字那里。**)