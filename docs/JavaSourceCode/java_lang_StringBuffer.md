

# java.lang.StringBuffer 阅读总结

## 概要

StringBuffer继承了AbstractStringBuilder抽象类，它与StringBuilder几乎是一样的，除了他们的安全性以及由此导致的速度问题(**StringBuffer是线程安全的，但是他的速度比StringBuilder慢了10~20%**)。StringBuffer以及StringBuilder是为了解决String一旦修改就要产生新对象的问题而产生的。

它有以下属性：

- char[] value //用于存储字符串，继承自AbstractStringBuilder
- int count// 存储字符串的的个数，继承自AbstractStringBuilder
- transient char[] toStringCache // toString的缓存，任何改变都会清空他，同时transient修饰会使得它不参与序列化

## 方法

### 构造方法

构造方法没有太大变化。

```java
    /**
    * 默认value容量为 16
    */
	public StringBuffer() {
        super(16);
    }

	/**
	* 指定value容量为capacity
	* capacity 为负数抛出 NegativeArraySizeException 异常
	*/
    public StringBuffer(int capacity) {
        super(capacity);
    }

	/**
	* 以String 进行初始化并增加16的容量
	*/
    public StringBuffer(String str) {
        super(str.length() + 16);
        append(str);
    }
	
	/**
	* 以CharSequence 进行初始化并增加16的容量
	*/
	public StringBuffer(CharSequence seq) {
        this(seq.length() + 16);
        append(seq);
    }
```

### 其他方法

虽然方法都是重写了，加了**synchronized**修饰符(**大部分的修改方法**)，实际上就是方法体里还是调用的**AbstractStringBuilder的方法**，主要变化就是方法体里添加了对toStringCache的清除以及缓存。

**某些方法修改了字符串内容的方法可能没有添加synchronized关键字修饰，这是因为它方法里调用的方法加了synchronized修饰，如下：**

```java
	/**
     * @throws StringIndexOutOfBoundsException {@inheritDoc}
     */
    @Override
    public StringBuffer insert(int offset, int i) {
        // 从下面可以看出来该方法会调用insert(int offset, String str) 这个方法
        // 而这个方法是使用了synchronized关键字修饰了的
        // 因此此方法也是线程安全的
        super.insert(offset, i);
        return this;
    }

	public AbstractStringBuilder insert(int offset, int i) {
        return insert(offset, String.valueOf(i));
    }

    @Override
    public synchronized StringBuffer insert(int offset, String str) {
        toStringCache = null;
        super.insert(offset, str);
        return this;
    }
```

#### String toString()

需要注意的是toString返回之前会先缓存，即填入到**toStringCache**。

#### 序列化

以下三个方法是为了实现StringBuffer的序列化(**将对象写到流中**)与反序列化(**从流中恢复对象**)。

[Java 序列化](https://juejin.im/post/5ce3cdc8e51d45777b1a3cdf)

```java
    /**
     * Serializable fields for StringBuffer.
     *
     * @serialField value  char[]
     *              The backing character array of this StringBuffer.
     * @serialField count int
     *              The number of characters in this StringBuffer.
     * @serialField shared  boolean
     *              A flag indicating whether the backing array is shared.
     *              The value is ignored upon deserialization.
     */
    private static final java.io.ObjectStreamField[] serialPersistentFields =
    {
        new java.io.ObjectStreamField("value", char[].class),
        new java.io.ObjectStreamField("count", Integer.TYPE),
        new java.io.ObjectStreamField("shared", Boolean.TYPE),
    };

    /**
     * 序列化
     */
    private synchronized void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        java.io.ObjectOutputStream.PutField fields = s.putFields();
        fields.put("value", value);
        fields.put("count", count);
        fields.put("shared", false);
        s.writeFields();
    }

    /**
     * 反序列化
     */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        java.io.ObjectInputStream.GetField fields = s.readFields();
        value = (char[])fields.get("value", null);
        count = fields.get("count", 0);
    }
```

