# java.lang.AbstractStringBuilder 阅读总结

## 概述

AbstractStringBuilder类是一个抽象类，它实现了Appendable(**可添加字符**)以及CharSequence(**可读的字符序列**)接口。它是为了解决String类每次进行修改就创建新对象的问题，具体实现有StringBuilder以及StringBuffer，前者是线程非安全的，后者是线程安全的，两者的区别主要在于StringBuffer的方法大部分用synchronized修饰了。

从源码中可以看出来，AbstractStringBuilder类和String类是非常相似的，设计的思想可以说是一模一样，其内部维护的都是一个char类的数组。唯一的区别在于这个数组的修饰符不一样，String类维护的是一个不可变的数组，而后者维护的则是一个可变的数组(**用final修饰**)。

它有以下属性：

- char[] value //用于存储字符串
- int count// 存储字符串的的个数

## 方法

### 构造方法

```java
    /**
     * 此无参数构造函数对于子类的序列化是必需的。
     */
    AbstractStringBuilder() {
    }

    /**
     * 创建指定容量的AbstractStringBuilder。
     */
    AbstractStringBuilder(int capacity) {
        value = new char[capacity];
    }
```

### 其他方法

#### int length()

返回已有字符串的长度，该值为count而非value的大小。

#### int capacity()

返回字符数组的容量，即value的长度。如果容量不够是发生添加会进行一个扩容，见expandCapacity(int miniumCapacity)方法，如下:

```java
	/**
     * 从下面代码可以看出，新容量是 原容量*2 + 2 与 minimumCapacity 的较大值
     */
    void expandCapacity(int minimumCapacity) {
        int newCapacity = value.length * 2 + 2;
        if (newCapacity - minimumCapacity < 0)
            newCapacity = minimumCapacity;
        if (newCapacity < 0) {
            if (minimumCapacity < 0) // overflow
                throw new OutOfMemoryError();
            newCapacity = Integer.MAX_VALUE;
        }
        value = Arrays.copyOf(value, newCapacity);
    }
```

#### void ensureCapacity(int minimumCapacity)

确保容量至少等于指定的最小值。如果当前容量小于该参数，则使用新的内部容量为数组分配更大的容量。

新容量是 原容量*2 + 2 与 minimumCapacity 的较大值。调用ensureCapacityInternal(int minimumCapacity)方法。

#### void ensureCapacityInternal(int minimumCapacity)

与上面方法ensureCapacity(int minimumCapacity)一样，由它调用该方法。

最终由expandCapacity(int minimumCapacity)方法实现。

#### void trimToSize() 

尝试减少用于字符序列的存储空间(**即设value数组的长度为count**)。如果缓冲区(value.length)大于保存当前字符序列所需的大小(count)，则可以调整缓冲区大小以提高空间效率。调用此方法可能(但不是一定)影响后续调用capacity()方法返回的值。

```java
	public void trimToSize() {
        if (count < value.length) {
            value = Arrays.copyOf(value, count);
        }
    }
```

#### void setLength(int newLength)

设置count的值为newLength，其中，如果

- newLength < 0，抛出IndexOutOfBoundsException异常。

- count < newLength，索引小于count的符号不变，大于count小于newLength的置为'/0'。
- count > newLength，进行裁剪。

#### char charAt(int index)

返回索引为index的字符，如果index<0 || >count ，抛出IndexOutOfBoundsException异常。

#### void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin)

依次拷贝value数组里从 srcBegin 开始到 srcEnd 的字符到 dst 从 dstBegin 开始的位置。

任何超出范围如srcEndg > count，srcEnd - srcBegin + dstBegin > dst.length 会抛出IndexOutOfBoundException异常。

#### void setCharAt(int index, char ch)

设定指定索引 index 位置的字符为 ch，如果index<0 || >count ，抛出IndexOutOfBoundsException异常。

#### AbstractStringBuilder append(String str)

将str添加到value数组里去，首先执行容量检测，其次调用getChars方法，最后更新count。

**如果str参数为空会添加"null"到value数组里。**

#### AbstractStringBuilder append(boolean b)

如果 b 为true，添加"true"到value数组，如果 b 为false，添加"false"到value数组。

#### AbstractStringBuilder append(int i)

下面源码可以看出，在加入整型时，最小值是直接写成字符串加入的，这是因为在之后有一个对 i 的取反操作。

```java
	AbstractStringBuilder append(int i) {
        if (i == Integer.MIN_VALUE) {
            append("-2147483648");
            return this;
        }
        // 如果是负数这里取反了
        int appendedLength = (i < 0) ? Integer.stringSize(-i) + 1
                                     : Integer.stringSize(i);
        int spaceNeeded = count + appendedLength;
    	// 先检查容量，不够就扩容
        ensureCapacityInternal(spaceNeeded);
        // 调用的是Integer的getChars方法
        Integer.getChars(i, spaceNeeded, value);
        count = spaceNeeded;
        return this;
    }

	// 与 int 相似操作
	public AbstractStringBuilder append(long l) {
        if (l == Long.MIN_VALUE) {
            append("-9223372036854775808");
            return this;
        }
        // 如果是负数这里取反了
        int appendedLength = (l < 0) ? Long.stringSize(-l) + 1
                                     : Long.stringSize(l);
        int spaceNeeded = count + appendedLength;
        // 先检查容量，不够就扩容
        ensureCapacityInternal(spaceNeeded);
        // 调用的是Long的getChars方法
        Long.getChars(l, spaceNeeded, value);
        count = spaceNeeded;
        return this;
    }
```

#### 其他几个append方法

```java
  	// 其他几个append方法

	 public AbstractStringBuilder append(StringBuffer sb) {
        if (sb == null)
            return appendNull();
        int len = sb.length();
        ensureCapacityInternal(count + len);
        sb.getChars(0, len, value, count);
        count += len;
        return this;
    }

    /**
     * @since 1.8
     */
    AbstractStringBuilder append(AbstractStringBuilder asb) {
        if (asb == null)
            return appendNull();
        int len = asb.length();
        ensureCapacityInternal(count + len);
        asb.getChars(0, len, value, count);
        count += len;
        return this;
    }

	@Override
    public AbstractStringBuilder append(CharSequence s, int start, int end) {
        if (s == null)
            s = "null";
        if ((start < 0) || (start > end) || (end > s.length()))
            throw new IndexOutOfBoundsException(
                "start " + start + ", end " + end + ", s.length() "
                + s.length());
        int len = end - start;
        ensureCapacityInternal(count + len);
        for (int i = start, j = count; i < end; i++, j++)
            value[j] = s.charAt(i);
        count += len;
        return this;
    }

	public AbstractStringBuilder append(char[] str) {
        int len = str.length;
        ensureCapacityInternal(count + len);
        System.arraycopy(str, 0, value, count, len);
        count += len;
        return this;
    }
	
	public AbstractStringBuilder append(char str[], int offset, int len) {
        if (len > 0)                // let arraycopy report AIOOBE for len < 0
            ensureCapacityInternal(count + len);
        System.arraycopy(str, offset, value, count, len);
        count += len;
        return this;
    }

	@Override
    public AbstractStringBuilder append(char c) {
        ensureCapacityInternal(count + 1);
        value[count++] = c;
        return this;
    }

    public AbstractStringBuilder append(float f) {
        FloatingDecimal.appendTo(f,this);
        return this;
    }

    public AbstractStringBuilder append(double d) {
        FloatingDecimal.appendTo(d,this);
        return this;
    }
```

#### AbstractStringBuilder delete(int start, int end)

删除从start开始到end结束的字符，包括start，不包括end。通过 system.arraycopy() 方法实现。

如果start == end 将不会删除任何字符。

#### AbstractStringBuilder deleteCharAt(int index)

删除指定位置的字符，通过 system.arraycopy()  方法前移一位实现。

#### AbstractStringBuilder replace(int start, int end, String str)

使用str替换从start(**包括**)开始到end(**不包括**)结束的子字符串。

流程：

- 判断是否越界；
- 对value 的容量进行一个检验；
- 然后将end之后的字符拷贝到start + str.length() 的位置；
- 最后将str拷贝到value的start；
- 更新count；

#### String substring(int start, int end)

取得从start(**包括**)开始到end(**不包括**)结束的子字符串。

String substring(int start) 调用该方法。

#### AbstractStringBuilder insert(int index, char[] str, int offset, int len)

将str数组从offset开始的长度为len的字符插入到该value从index开始的位置。

流程：

- 判断是否存在越界；
- 对value 容量的一个检验；
- 移动value 从index开始到count的字符到value的index + len 的位置；
- 添加str数组到value数组index开始的位置；
- 更新count；

#### AbstractStringBuilder insert(int offset, String str)

在value数组的offset位置添加str字符串。

流程：

- 判断是否存在数组越界；
- 如果str==null，直接加上"null"；
- 对value容量的一个检验；
- 移动从offset开始的字符到value的offset + str.length()的位置；
- 复制str到value的offset到offset + str.length()；
- 更新count；

```java
	public AbstractStringBuilder insert(int offset, String str) {
        if ((offset < 0) || (offset > length()))
            throw new StringIndexOutOfBoundsException(offset);
        if (str == null)
            str = "null";
        int len = str.length();
        ensureCapacityInternal(count + len);
        System.arraycopy(value, offset, value, offset + len, count - offset);
        str.getChars(value, offset);
        count += len;
        return this;
    }

    public AbstractStringBuilder insert(int offset, char[] str) {
        if ((offset < 0) || (offset > length()))
            throw new StringIndexOutOfBoundsException(offset);
        int len = str.length;
        ensureCapacityInternal(count + len);
        System.arraycopy(value, offset, value, offset + len, count - offset);
        System.arraycopy(str, 0, value, offset, len);
        count += len;
        return this;
    }

    public AbstractStringBuilder insert(int dstOffset, CharSequence s) {
        if (s == null)
            s = "null";
        if (s instanceof String)
            return this.insert(dstOffset, (String)s);
        return this.insert(dstOffset, s, 0, s.length());
    }

     public AbstractStringBuilder insert(int dstOffset, CharSequence s,
                                         int start, int end) {
        if (s == null)
            s = "null";
        if ((dstOffset < 0) || (dstOffset > this.length()))
            throw new IndexOutOfBoundsException("dstOffset "+dstOffset);
        if ((start < 0) || (end < 0) || (start > end) || (end > s.length()))
            throw new IndexOutOfBoundsException(
                "start " + start + ", end " + end + ", s.length() "
                + s.length());
        int len = end - start;
        ensureCapacityInternal(count + len);
        System.arraycopy(value, dstOffset, value, dstOffset + len,
                         count - dstOffset);
        for (int i=start; i<end; i++)
            value[dstOffset++] = s.charAt(i);
        count += len;
        return this;
    }

    public AbstractStringBuilder insert(int offset, boolean b) {
        // b 为true，插入"true" false 插入"false"
        return insert(offset, String.valueOf(b));
    }

	// 扩容之后直接加
    public AbstractStringBuilder insert(int offset, char c) {
        ensureCapacityInternal(count + 1);
        System.arraycopy(value, offset, value, offset + 1, count - offset);
        value[offset] = c;
        count += 1;
        return this;
    }

    public AbstractStringBuilder insert(int offset, int i) {
        return insert(offset, String.valueOf(i));
    }

    public AbstractStringBuilder insert(int offset, long l) {
        return insert(offset, String.valueOf(l));
    }

    public AbstractStringBuilder insert(int offset, float f) {
        return insert(offset, String.valueOf(f));
    }

    public AbstractStringBuilder insert(int offset, double d) {
        return insert(offset, String.valueOf(d));
    }
```

#### int indexOf(String str)

找到value中匹配str的位置(第一个字符位置)。调用indexOf(String str, int fromIndex)方法。

#### int indexOf(String str, int fromIndex)

找到从fromIndex开始value中匹配str的位置(第一个字符位置)。调用String.indexOf(value, 0, count, str, fromIndex)。

#### int lastIndexOf(String str)

找到最后一个匹配str的位置(第一个字符位置)。调用lastIndexOf(str, count)方法。

#### int lastIndexOf(String str, int fromIndex)

找到从fromIndex 开始最后一个匹配str的位置(第一个字符位置)。调用 String.lastIndexOf(value, 0, count, str, fromIndex)。

#### AbstractStringBuilder reverse()

反转字符串。使该字符序列替换为序列的反面。如果序列中包含任何代理项对，则将它们作为单个字符处理，以便进行反向操作。因此，高低代位的顺序永远不会颠倒。

值得注意的是，反向操作可能会产生未配对的低代理和高代理。例如,扭转"\u005CuDC00\u005CuD800" 生成"\u005CuD800\u005CuDC00"， 这是有效的代理项对。

```java
	public AbstractStringBuilder reverse() {
        boolean hasSurrogates = false;
        int n = count - 1;
        // 从中间开始，两边替换
        // 这里设置成(n-1)/2可以避免这奇数的情况下交换中心元素自身
        for (int j = (n-1) >> 1; j >= 0; j--) {
            int k = n - j;
            // 交换
            char cj = value[j];
            char ck = value[k];
            value[j] = ck;
            value[k] = cj;
            // 包含任何代理项对
            if (Character.isSurrogate(cj) ||
                Character.isSurrogate(ck)) {
                hasSurrogates = true;
            }
        }
        if (hasSurrogates) {
            reverseAllValidSurrogatePairs();
        }
        return this;
    }
	// 将辅平面的编码逆转回来
    private void reverseAllValidSurrogatePairs() {
        for (int i = 0; i < count - 1; i++) {
            char c2 = value[i];
            if (Character.isLowSurrogate(c2)) {
                char c1 = value[i + 1];
                if (Character.isHighSurrogate(c1)) {
                    value[i++] = c1;
                    value[i] = c2;
                }
            }
        }
    }
```

#### abstract String toString()

返回表示此序列中数据的字符串。一个新的String对象被分配并初始化为包含当前由此字符表示的字符序列对象。然后返回此Stirng 对象。后续的更改此顺序不会影响内容。