# java.lang.Integer 阅读总结

## 概要

Integer 类是int的包装类，它继承了Number类，实现了Comparable接口。

Integer 类是对int进行封装,里面包含处理int类型的方法，比如int到String类型的转换方法或String类型转int类型的方法，也有与其他类型之间的转换方，当然也包括操作位的方法。

> 非new生成的Integer变量指向的是java常量池中的对象，而new Integer()生成的变量指向堆中新建的对象，两者在内存中的地址不同）

## 主要属性

Integer的范围是 **0x7fffffff`~`0x80000000**，与int的范围是一样的。在Integer类中这样声明它们：

```java
    // 最小值
	@Native public static final int   MIN_VALUE = 0x80000000;

	// 最大值
    @Native public static final int   MAX_VALUE = 0x7fffffff;

```

Integer对应的基础数据类型是int:

```java
	public static final Class<Integer>  TYPE = (Class<Integer>) Class.getPrimitiveClass("int");
```

Integer不同进制情况下可能出现的字符：

```java
	// 可能出现的位数
	final static char[] digits = {
        '0' , '1' , '2' , '3' , '4' , '5' ,
        '6' , '7' , '8' , '9' , 'a' , 'b' ,
        'c' , 'd' , 'e' , 'f' , 'g' , 'h' ,
        'i' , 'j' , 'k' , 'l' , 'm' , 'n' ,
        'o' , 'p' , 'q' , 'r' , 's' , 't' ,
        'u' , 'v' , 'w' , 'x' , 'y' , 'z'
    };
```

DigitTens和DigitOnes两个数组，DigitTens数组是为了获取0到99之间某个数的十位,DigitOnes是为了获取0到99之间某个数的个位。

```java
	final static char [] DigitTens = {
        '0', '0', '0', '0', '0', '0', '0', '0', '0', '0',
        '1', '1', '1', '1', '1', '1', '1', '1', '1', '1',
        '2', '2', '2', '2', '2', '2', '2', '2', '2', '2',
        '3', '3', '3', '3', '3', '3', '3', '3', '3', '3',
        '4', '4', '4', '4', '4', '4', '4', '4', '4', '4',
        '5', '5', '5', '5', '5', '5', '5', '5', '5', '5',
        '6', '6', '6', '6', '6', '6', '6', '6', '6', '6',
        '7', '7', '7', '7', '7', '7', '7', '7', '7', '7',
        '8', '8', '8', '8', '8', '8', '8', '8', '8', '8',
        '9', '9', '9', '9', '9', '9', '9', '9', '9', '9',
        } ;

    final static char [] DigitOnes = {
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        } ;
```

sizeTable 则是为了取得用String表示的int的位数，如下：

```java
	final static int [] sizeTable = { 9, 99, 999, 9999, 99999, 999999, 9999999,
                                      99999999, 999999999, Integer.MAX_VALUE };

    // Requires positive x
    static int stringSize(int x) {
        for (int i=0; ; i++)
            if (x <= sizeTable[i])
                return i+1;
    }
```

value 属性用来存储当前值，size用来标识int的位数（32），BYTES用来标识int的字节数，serialVersionUID序列化。

```java

	private final int value;

	/**
     * 位数
     */
    @Native public static final int SIZE = 32;

    /**
     * 字节数
     */
    public static final int BYTES = SIZE / Byte.SIZE;

	@Native private static final long serialVersionUID = 1360826667806852920L;
```

### IntegerCache内部类

```java
private static class IntegerCache {
    
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```

IntegerCache用一个Integer数组来存储一些Integer对象，一共有(high-low)个，即(127+128)个。当Integer的值范围在[-128,127]之间时，不必重新实例化，直接从缓存中获取。这些缓存值都是静态且final的，避免重复的实例化和回收。

>如果不去配置虚拟机参数，high这个值不会变。配合valueOf(int) 方法，可以节省创建对象造成的资源消耗。另外如果想改变这些值缓存的范围，再启动JVM时可以通过-Djava.lang.Integer.IntegerCache.high=xxx就可以改变缓存值的最大值，比如-Djava.lang.Integer.IntegerCache.high=888则会缓存[-888]。 

## 方法

### 构造方法

两个，一个传int，一个传String。第二种调用parseInt方法进行转换。

```java

    public Integer(int value) {
        this.value = value;
    }

    /**
     * 将String类型的s用10进制进行转换，转换失败抛出NumberFormatException
     */
    public Integer(String s) throws NumberFormatException {
        this.value = parseInt(s, 10);
    }
```

### 其他方法

#### getChars(int i, int index, char[] buf)

将数字i以char的形式从index开始倒序依次放入buf。

```java
	// i = 123456789,buf = [1,2,3,4,5,6,7,8,9]
	static void getChars(int i, int index, char[] buf) {
        int q, r;
        int charPos = index;
        char sign = 0;

        if (i < 0) {
            sign = '-';
            i = -i;
        }

        // 每次操作取个位和十位的数字填充
        // while (i >= 65536)部分就是处理高位的两个字节，大于65536的部分使用了除法，一次生成两位十进制字符，每次处理2位数，其实((q << 6) + (q << 5) + (q << 2))其实等于q*100,DigitTens和DigitOnes数组前面已经讲过它的作用了，用来获取十位和个位。
        while (i >= 65536) {
            q = i / 100;
        	// really: r = i - (q * 100);
            r = i - ((q << 6) + (q << 5) + (q << 2));
            i = q;
            buf [--charPos] = DigitOnes[r];// 个位
            buf [--charPos] = DigitTens[r];// 十位
        }

        //小于等于65536的部分使用了乘法加位移做成除以10的效果，比如(i * 52429) >>> (16+3)其实约等于i/10，((q << 3) + (q << 1))其实等于q*10，然后再通过digits数组获取到对应的字符。可以看到低位处理时它尽量避开了除法，取而代之的是用乘法和右移来实现，可见除法比起乘法和移位是一个比较耗时的操作。
        for (;;) {
            q = (i * 52429) >>> (16+3);
            r = i - ((q << 3) + (q << 1));  // r = i-(q*10) ...
            buf [--charPos] = digits [r];
            i = q;
            if (i == 0) break;
        }
        if (sign != 0) {
            buf [--charPos] = sign;
        }
    }
```

#### toString()

```java
    public String toString() {
        return toString(value);
    }

	public static String toString(int i) {
        // 单独处理是因为stringSize里进行转换越界了，stringSize代码见上面属性介绍
        if (i == Integer.MIN_VALUE)
            return "-2147483648";
        // 负数要+1是因为“-”
        int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);
        char[] buf = new char[size];
        // 按数字的顺序放置在buf里面
        getChars(i, size, buf);
        return new String(buf, true);
    }

	// 它会转换成对应进制的字符串。不在2到36进制范围之间的都会按照10进制处理。	
    public static String toString(int i, int radix) {
        // 进制超出范围选用10进制
        if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
            radix = 10;

        if (radix == 10) {
            return toString(i);
        }
	
        // 申请33位是因为最大32位加一个符号位
        char buf[] = new char[33];
        boolean negative = (i < 0);
        int charPos = 32;

        // 正数负数统一处理
        if (!negative) {
            i = -i;
        }

        while (i <= -radix) {
            // 填充个位数，digits在上面有介绍
            buf[charPos--] = digits[-(i % radix)];
            i = i / radix;
        }
        buf[charPos] = digits[-i];

        if (negative) {
            buf[--charPos] = '-';
        }

        return new String(buf, charPos, (33 - charPos));
    }
```

#### toHexString和toOctalString方法

```java
	public static String toHexString(int i) {
        return toUnsignedString0(i, 4);
    }
	public static String toOctalString(int i) {
        return toUnsignedString0(i, 3);
    }
	public static String toBinaryString(int i) {
        return toUnsignedString0(i, 1);
    }
	private static String toUnsignedString0(int val, int shift) {
        // 计算需要的字符数
        int mag = Integer.SIZE - Integer.numberOfLeadingZeros(val);
        int chars = Math.max(((mag + (shift - 1)) / shift), 1);
        char[] buf = new char[chars];

        formatUnsignedInt(val, shift, buf, 0, chars);

        return new String(buf, true);
    }

	static int formatUnsignedInt(int val, int shift, char[] buf, int offset, int len) {
        int charPos = len;
        int radix = 1 << shift;
        int mask = radix - 1;
        do {
            // val & mask 取得该进制的个位数
            buf[offset + --charPos] = Integer.digits[val & mask];
            val >>>= shift;
        } while (val != 0 && charPos > 0);

        return charPos;
    }
```

这两个方法类似，合到一起讲。看名字就知道转成8进制和16进制的字符串。可以看到都是间接调用toUnsignedString0方法，该方法会先计算转换成对应进制需要的字符数，然后再通过formatUnsignedInt方法来填充字符数组，该方法做的事情就是使用进制之间的转换方法（前面有提到过）来获取对应的字符。

#### decode方法

```java
public static Integer decode(String nm) throws NumberFormatException {
    int radix = 10;
    int index = 0;
    boolean negative = false;
    Integer result;

    if (nm.length() == 0)
        throw new NumberFormatException("Zero length string");
    char firstChar = nm.charAt(0);
    // Handle sign, if present
    if (firstChar == '-') {
        negative = true;
        index++;
    } else if (firstChar == '+')
        index++;

    // Handle radix specifier, if present
    if (nm.startsWith("0x", index) || nm.startsWith("0X", index)) {
        index += 2;
        radix = 16;
    }
    else if (nm.startsWith("#", index)) {
        index ++;
        radix = 16;
    }
    else if (nm.startsWith("0", index) && nm.length() > 1 + index) {
        index ++;
        radix = 8;
    }

    if (nm.startsWith("-", index) || nm.startsWith("+", index))
        throw new NumberFormatException("Sign character in wrong position");

    try {
        result = Integer.valueOf(nm.substring(index), radix);
        result = negative ? Integer.valueOf(-result.intValue()) : result;
    } catch (NumberFormatException e) {
        // If number is Integer.MIN_VALUE, we'll end up here. The next line
        // handles this case, and causes any genuine format error to be
        // rethrown.
        String constant = negative ? ("-" + nm.substring(index))
                                   : nm.substring(index);
        result = Integer.valueOf(constant, radix);
    }
    return result;
}复制代码
```

decode方法主要作用是对字符串进行解码。

默认会处理成十进制，0开头的都会处理成8进制，0x和#开头的都会处理成十六进制。

#### xxx value方法（byteValue，shortValue，intValue，longValue，floatValue，doubleValue）

```java
	public byte byteValue() {
        return (byte)value;
    }
	public short shortValue() {
        return (short)value;
    }
	public int intValue() {
        return value;
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

转换成对应的类型。

#### hashCode方法与equals方法

hashCode方法就是返回int类型的值。

```java
    public int hashCode() {
        return value;
    }
```

equals 方法比较是否相同之前会将int类型通过valueof转换成Integer类型，equals本质就是值得比较。

```java
    public boolean equals(Object obj) {
        if (obj instanceof Integer) {
            return value == ((Integer)obj).intValue();
        }
        return false;
    }
```

#### compare方法

x小于y则返回-1，相等则返回0，否则返回1。

```java
	public static int compare(int x, int y) {      
   		return (x < y) ? -1 : ((x == y) ? 0 : 1);
	}
```

#### bitCount方法

```java
   public static int bitCount(int i) {
        // HD, Figure 5-2
        i = i - ((i >>> 1) & 0x55555555);
        i = (i & 0x33333333) + ((i >>> 2) & 0x33333333);
        i = (i + (i >>> 4)) & 0x0f0f0f0f;
        i = i + (i >>> 8);
        i = i + (i >>> 16);
        return i & 0x3f;
    }

```

该方法主要用于计算二进制数中1的个数。

这些都是移位和加减操作。

`0x55555555`等于`01010101010101010101010101010101`，`0x33333333`等于`110011001100110011001100110011`，`0x0f0f0f0f`等于`1111000011110000111100001111`。

先每**两位**一组统计看有多少个1，比如`10011111`则每两位有1、1、2、2个1，记为`01011010`，然后再算每**四位**一组看有多少个1，而`01011010`则每四位有2、4个1，记为`00100100`，接着每**八位**一组就为`00000110`，接着**16位，32位**，最终在与`0x3f`进行与运算，得到的数即为1的个数。

#### highestOneBit方法

```java
    public static int highestOneBit(int i) {
        // HD, Figure 3-1
        i |= (i >>  1);
        i |= (i >>  2);
        i |= (i >>  4);
        i |= (i >>  8);
        i |= (i >> 16);
        return i - (i >>> 1);
    }
// 随便一个例子，不用管最高位之后有多少个1，都会被覆盖
// 00010000 00000000 00000000 00000000      raw
// 00011000 00000000 00000000 00000000      i | (i >> 1)
// 00011110 00000000 00000000 00000000      i | (i >> 2)
// 00011111 11100000 00000000 00000000      i | (i >> 4)
// 00011111 11111111 11100000 00000000      i | (i >> 8)
// 00011111 11111111 11111111 11111111      i | (i >> 16)
// 00010000 0000000 00000000 00000000       i - (i >>>1)复制代码
```

这个方法是返回最高位的1，其他都是0的值。将i右移一位再或操作，则最高位1的右边也为1了，接着再右移两位并或操作，则右边1+2=3位都为1了，接着1+2+4=7位都为1，直到1+2+4+8+16=31都为1，最后用`i - (i >>> 1)`自然得到最终结果。

#### lowestOneBit方法

```java
    public static int lowestOneBit(int i) {
        // HD, Section 2-1
        return i & -i;
    }
// 例子
// 00001000 10000100 10001001 00101000    i
// 11110111 01111011 01110110 11011000    -i
// 00000000 00000000 00000000 00001000  i & -i
```

与highestOneBit方法对应，lowestOneBit获取最低位1，其他全为0的值。这个操作较简单，先取负数，这个过程需要对正数的i取反码然后再加1，得到的结果和i进行与操作，刚好就是最低位1其他为0的值了。

#### numberOfLeadingZeros方法

```java
    public static int numberOfLeadingZeros(int i) {
        // HD, Figure 5-6
        if (i == 0)
            return 32;
        int n = 1;
        if (i >>> 16 == 0) { n += 16; i <<= 16; }
        if (i >>> 24 == 0) { n +=  8; i <<=  8; }
        if (i >>> 28 == 0) { n +=  4; i <<=  4; }
        if (i >>> 30 == 0) { n +=  2; i <<=  2; }
        n -= i >>> 31;
        return n;
    }
// 方法很巧妙， 类似于二分法。不断将数字左移缩小范围。例子用最差情况：
// i: 00000000 00000000 00000000 00000001         n = 1
// i: 00000000 00000001 00000000 00000000         n = 17
// i: 00000001 00000000 00000000 00000000         n = 25
// i: 00010000 00000000 00000000 00000000         n = 29
// i: 01000000 00000000 00000000 00000000         n = 31
// i >>>31 == 0
// return 31
```

该方法返回i的二进制从头开始有多少个0。i为0的话则有32个0。这里处理其实是体现了二分查找思想的，先看高16位是否为0，是的话则至少有16个0，否则左移16位继续往下判断，接着右移24位看是不是为0，是的话则至少有16+8=24个0，直到最后得到结果。

#### numberOfTrailingZeros方法

```java
    public static int numberOfTrailingZeros(int i) {
        // HD, Figure 5-14
        int y;
        if (i == 0) return 32;
        int n = 31;
        y = i <<16; if (y != 0) { n = n -16; i = y; }
        y = i << 8; if (y != 0) { n = n - 8; i = y; }
        y = i << 4; if (y != 0) { n = n - 4; i = y; }
        y = i << 2; if (y != 0) { n = n - 2; i = y; }
        return n - ((i << 1) >>> 31);
    }
// 与求开头多少个0类似，也是用了二分法，先锁定1/2, 再锁定1/4，1/8，1/16，1/32。
// i: 11111111 11111111 11111111 11111111    n: 31
// i: 11111111 11111111 00000000 00000000    n: 15
// i: 11111111 00000000 00000000 00000000    n: 7
// i: 11110000 00000000 00000000 00000000    n: 3
// i: 11000000 00000000 00000000 00000000    n: 1
// i: 10000000 00000000 00000000 00000000    n: 0
```

与前面的numberOfLeadingZeros方法对应，该方法返回i的二进制从尾开始有多少个0。它的思想和前面的类似，也是基于二分查找思想，详细步骤不再赘述。

#### reverse方法

```java
	public static int reverse(int i) {
        i = (i & 0x55555555) << 1 | (i >>> 1) & 0x55555555;
        i = (i & 0x33333333) << 2 | (i >>> 2) & 0x33333333;
        i = (i & 0x0f0f0f0f) << 4 | (i >>> 4) & 0x0f0f0f0f;
        i = (i << 24) | ((i & 0xff00) << 8) |
            ((i >>> 8) & 0xff00) | (i >>> 24);
        return i;
    }
// 0x55555555  01010101 01010101 01010101 01010101
// 0x33333333  00110011 00110011 00110011 00110011
// 0x0f0f0f0f  11110000 11110000 11110000 11110000
// 0xff00      00000000 00000000 11111111 00000000
// i: 00000000 00000000 00000000 10100111 开始
// i: 00000000 00000000 00000000 01010111 交换相邻两位
// i: 00000000 00000000 00000000 01011101 交换相邻四位
// i: 00000000 00000000 00000000 11010101 交换相邻八位
// i: 11010101 00000000 00000000 00000000 最高八位与最低八位交换，中间16位交换
```

该方法即是将i进行反转，反转就是第1位与第32位对调，第二位与第31位对调，以此类推。

