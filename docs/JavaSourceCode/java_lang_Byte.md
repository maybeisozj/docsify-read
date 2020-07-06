# java.lang.Byte 阅读总结

## 概要

Byte 是字节(byte)的包装类，最大127，最小-128（它有两个静态实例变量MAX_VALUE以及MIN_VALUE分别代表最大值127与最小值-128）。它继承了Number类并实现了Comparable接口，它还提供了与String类型转换的相关函数。

> 字节类型(byte)是Java中可用的最小整数数据类型。当程序使用其值在`-128`到`127`范围内的大量变量或在文件或网络中处理二进制数据时，使用字节变量。字节类型没有字节字面量。可以将任何在字节范围内的`int`字面量分配给一个字节变量。

它有一个value来存储对应的byte值。相关属性如下:

```java
    /**
     * 最小值
     */
    public static final byte   MIN_VALUE = -128;

    /**
     * 最大值
     */
    public static final byte   MAX_VALUE = 127;

    /**
     * 包装class里的byte数据类型
     */
    @SuppressWarnings("unchecked")
    public static final Class<Byte>     TYPE = (Class<Byte>) Class.getPrimitiveClass("byte");

	/**
     * 存储Byte实例的值
     */
    private final byte value;

	/**
     * 位数
     */
    public static final int SIZE = 8;
    
    public static final int BYTES = SIZE / Byte.SIZE;

    /** 序列化 */
    private static final long serialVersionUID = -7183698231559129828L;
```
内部静态类ByteCache：

```java
private static class ByteCache {
    private ByteCache(){}

    static final Byte cache[] = new Byte[-(-128) + 127 + 1];

    static {
        for(int i = 0; i < cache.length; i++)
            cache[i] = new Byte((byte)(i - 128));
    }
}
```

## 方法

### 构造方法

```java
    /**
     * 新建一个Byte对象来代表给定的value
     */
    public Byte(byte value) {
        this.value = value;
    }

    /**
     * 新建一个Byte对象来代表给定的10进制的String的值。
     * 如果String不能被转换则抛出NumberFormatException异常。
     */
    public Byte(String s) throws NumberFormatException {
        this.value = parseByte(s, 10);
    }
```

### 其他方法

#### String toString() 

返回代表该字节类型b的10进制值的String对象。

```java
    public static String toString(byte b) {
        return Integer.toString((int)b, 10);
    }

	public String toString() {
        return Integer.toString((int)value);
    }
```

#### static Byte valueOf(byte b)

返回一个值为b的Byte对象。看看源码我们就知道这里用的是上面提到过的ByteCache了。**如果不需要一个新的对象的话，推荐使用这个方法。**

```java
    public static Byte valueOf(byte b) {
        final int offset = 128;
        return ByteCache.cache[(int)b + offset];
    }
```

#### String的转换方法

```java
    /**
     * 基于选择的radix进制来转换String(**里面必须全是数字，+ ， - **)到byte，无法转换抛出
     * NumberFormatException异常。
     */
	public static byte parseByte(String s, int radix)
        throws NumberFormatException {
        int i = Integer.parseInt(s, radix);
        // 这里超过了Byte的范围
        if (i < MIN_VALUE || i > MAX_VALUE)
            throw new NumberFormatException(
                "Value out of range. Value:\"" + s + "\" Radix:" + radix);
        return (byte)i;
    }
	
	/**
	 * 以10进制来转换String得到byte。
	 */
	public static byte parseByte(String s) throws NumberFormatException {
        return parseByte(s, 10);
    }
	
	/**
	 * 调用parseByte转换得到byte类型之后调用valueOf方法得到对应实例。
	 */
	public static Byte valueOf(String s, int radix)
        throws NumberFormatException {
        return valueOf(parseByte(s, radix));
    }
	
	/**
	 * 默认10进制的valueOf(s, 10)返回结果。
	 */
	public static Byte valueOf(String s) throws NumberFormatException {
        return valueOf(s, 10);
    }

	/**
     * 解码String到byte。
     */
	public static Byte decode(String nm) throws NumberFormatException {
        int i = Integer.decode(nm);
        if (i < MIN_VALUE || i > MAX_VALUE)
            throw new NumberFormatException(
                    "Value " + i + " out of range from input " + nm);
        return valueOf((byte)i);
    }
```

#### 各种value

```java
    public byte byteValue() {
        return value;
    }

    public short shortValue() {
        return (short)value;
    }

    public int intValue() {
        return (int)value;
    }

    public long longValue() {
        return (long)value;
    }

    public float floatValue() {
        return (float)value;
    }

    public double doubleValue() {
        return (double)value;
    }
```

#### hashcode 与 Equals

```java
    @Override
    public int hashCode() {
        return Byte.hashCode(value);
    }

    /**
     * 可以看到Byte的hashcode值就是value的整型数值。
     */
    public static int hashCode(byte value) {
        return (int)value;
    }

    /**
     * 比较的是数值
     */
    public boolean equals(Object obj) {
        if (obj instanceof Byte) {
            return value == ((Byte)obj).byteValue();
        }
        return false;
    }
```

#### comparable相关

```java
    public int compareTo(Byte anotherByte) {
        return compare(this.value, anotherByte.value);
    }

    /**
     * 0 x == y;
     * 正数 x > y;
     * 负数 x < y;
     */
    public static int compare(byte x, byte y) {
        return x - y;
    }
```

#### 无符号转换

```java
    /**
     * byte 是1个字节的，int是4个字节，转换成无符号int之后，3个字节(高24位)为0，
     * 低8位与byte相等。
     * 0和正数的转换结果与(int)byte 转换没有差别，负数相当于加了2的8次方。
     */
    public static int toUnsignedInt(byte x) {
        return ((int) x) & 0xff;
    }

    /**
     * byte 是1个字节的，long是8个字节，转换成无符号int之后，7个字节(高56位)为0，
     * 低8位与byte相等。
     * 0和正数的转换结果与(int)byte 转换没有差别，负数相当于加了2的8次方。
     */
    public static long toUnsignedLong(byte x) {
        return ((long) x) & 0xffL;
    }
```