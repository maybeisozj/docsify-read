# java.lang.Double 阅读总结

## 概要

Double是double基础类型的包装类，它继承了Number类并实现了Comparable接口，它还提供了与String类型转换的相关函数。

它有正无穷，负无穷，NaN(不是数)，最大值，最小值，最小正标准值的常量，最小正非零值的常量，具体如下：

```java
	/**
     * double 的正无穷
     */
    public static final double POSITIVE_INFINITY = 1.0 / 0.0;

    /**
     * double 的负无穷
     */
    public static final double NEGATIVE_INFINITY = -1.0 / 0.0;

    /**
     * NaN常量（Not a Number）
     */
    public static final double NaN = 0.0d / 0.0;

    /**
     * 最大正有限值的常量
     * (2-2<sup>-52</sup>)&middot;2<sup>1023</sup>.
     * {@code 0x1.fffffffffffffP+1023} {@code Double.longBitsToDouble(0x7fefffffffffffffL)}.
     */
    public static final double MAX_VALUE = 0x1.fffffffffffffP+1023; // 1.7976931348623157e+308

    /**
     * 最小正标准值的常量 2<sup>-1022</sup>.  
     * {@code 0x1.0p-1022} {@code Double.longBitsToDouble(0x0010000000000000L)}.
     */
    public static final double MIN_NORMAL = 0x1.0p-1022; // 2.2250738585072014E-308

    /**
     * 最小正非零值的常量 2<sup>-1074</sup>. 
     * {@code 0x0.0000000000001P-1022} {@code Double.longBitsToDouble(0x1L)}.
     */
    public static final double MIN_VALUE = 0x0.0000000000001P-1022; // 4.9e-324

    /**
     * 可能具有的最大指数
     */
    public static final int MAX_EXPONENT = 1023;

    /**
     * 可能具有的最小指数
     */
    public static final int MIN_EXPONENT = -1022;

    /**
     *  double 值的位数
     */
    public static final int SIZE = 64;

    /**
     * The number of bytes used to represent a {@code double} value.
     *
     * @since 1.8
     */
    public static final int BYTES = SIZE / Byte.SIZE;

    /**
     * 基本类型 double 的 Class 实例
     */
    @SuppressWarnings("unchecked")
    public static final Class<Double>   TYPE = (Class<Double>) Class.getPrimitiveClass("double");
	
	/**
	 * double 值
	 */
	private final double value;

	/**
	 * 序列化
 	 */
	private static final long serialVersionUID = -9172774392245257468L;
```



## 方法

### 构造方法

```java
    /**
     * 为value分配Double对象实例
     */
    public Double(double value) {
        this.value = value;
    }

    /**
     * 为用String表示的double分配实例，如果无法转换抛出NumberFormatException
     */
    public Double(String s) throws NumberFormatException {
        value = parseDouble(s);
    }
```

### 其他方法

#### 静态方法

```java
    /**
     * 返回 double 参数的字符串表示形式。下面提到的所有字符都是 ASCII 字符。
     * 如果参数为 NaN，那么结果为字符串 "NaN"
     * 否则，结果是表示参数符号和数值（绝对值）的字符串。如果符号为负，那么结果的第一个字符是 '-'；
     * 如果符号为正，那么结果中不显示符号字符。
     	对于数值 m：
			如果 m 为无穷大，正无穷大生成结果 "Infinity"，负无穷大生成结果 "-Infinity"。
			如果 m 为 0，负 0 生成结果 "-0.0"，正 0 生成结果 "0.0"。
			如果 m 大于或者等于 10^-3，但小于 10^7，则采用不带前导 0 的十进制形式，用 m 的整数部分表示，后跟 '.' ，再后面是表示 m 小数部分的一个或多个十进制数字。
			如果 m 小于 10^-3 或大于等于 10^7，则使用所谓的“计算机科学记数法”表示。设 n 为满足 10^n <= m <10^n+1 的唯一整数；然后设 a 为 m 与 10^n 的精确算术商，从而 1 <= a < 10。那么，数值便表示为 a 的整数部分，其形式为：一个十进制数字，后跟 '.' ('\u002E')，接着是表示 a 小数部分的十进制数字，再后面是字母 'E' ('\u0045')，最后是用十进制整数形式表示的 n，这与方Integer.toString(int) 生成的结果一样。
	必须为 m 或 a 的小数部分显示多少位呢？至少必须有一位数来表示小数部分，除此之外，需要更多（但只能和需要的一样多）位数来唯一地区分参数值和 double 类型的邻近值。这就是说，假设 x 是用十进制表示法表示的精确算术值，是通过对有限非 0 参数 d 调用此方法生成的。那么 d 一定是最接近 x 的 double 值；如果两个 double 值都同等地接近 x，那么 d 必须是其中之一，并且 d 的有效数字的最低有效位必须是 0。
     */
    public static String toString(double d) {
        return FloatingDecimal.toJavaFormatString(d);
    }

    /**
     * 返回 double 参数的十六进制字符串表示形式。
        如果参数为 NaN，那么结果为字符串 "NaN"。
        否则，结果是表示参数符号和数值的字符串。如果符号为负，那么结果的第一个字符是 '-' ('\u002D')；如果符号为正，那么结果中不显示符号字符。
        对于数值 m：
            如果 m 为无穷大，正无穷大生成结果 "Infinity"，负无穷大生成结果 "-Infinity"。
            如果 m 为 0，负 0 生成结果 "-0x0.0p0"，正 0 生成结果 "0x0.0p0"。
            如果 m 是具有标准化表示形式的 double 值，则使用子字符串表示有效数字和指数字段。有效数字用字符 "0x1." 表示，后跟该有效数字剩余小数部分的小写十六进制表示形式。除非所有位数都为 0，否则移除十六进制表示中的尾部 0，在所有位数都为零的情况下，可以用一个 0 表示。接下来用 "p" 表示指数，后跟无偏指数的十进制字符串，该值与通过对指数值调用 Integer.toString 生成的值相同。
            如果 m 是非标准表示形式的 double 值，则用字符 "0x0." 表示有效数字，后跟该有效数字剩余小数部分的十六进制表示形式。移除十六进制表示中的尾部 0。然后用 "p-1022" 表示指数。注意，在非标准有效数字中，必须至少有一个非 0 数字。
        示例
        浮点值					十六进制字符串
        1.0						0x1.0p0
        -1.0					-0x1.0p0
        2.0						0x1.0p1
        3.0						0x1.8p1
        0.5						0x1.0p-1
        0.25					0x1.0p-2
        Double.MAX_VALUE		 0x1.fffffffffffffp1023
        Minimum Normal Value	  0x1.0p-1022
        Maximum Subnormal Value	  0x0.fffffffffffffp-1022
        Double.MIN_VALUE	      0x0.0000000000001p-1022
     */
    public static String toHexString(double d) {
        /*
         * Modeled after the "a" conversion specifier in C99, section
         * 7.19.6.1; however, the output of this method is more
         * tightly specified.
         */
        if (!isFinite(d) )
            // For infinity and NaN, use the decimal output.
            return Double.toString(d);
        else {
            // Initialized to maximum size of output.
            StringBuilder answer = new StringBuilder(24);

            if (Math.copySign(1.0, d) == -1.0)    // value is negative,
                answer.append("-");                  // so append sign info

            answer.append("0x");

            d = Math.abs(d);

            if(d == 0.0) {
                answer.append("0.0p0");
            } else {
                boolean subnormal = (d < DoubleConsts.MIN_NORMAL);

                // Isolate significand bits and OR in a high-order bit
                // so that the string representation has a known
                // length.
                long signifBits = (Double.doubleToLongBits(d)
                                   & DoubleConsts.SIGNIF_BIT_MASK) |
                    0x1000000000000000L;

                // Subnormal values have a 0 implicit bit; normal
                // values have a 1 implicit bit.
                answer.append(subnormal ? "0." : "1.");

                // Isolate the low-order 13 digits of the hex
                // representation.  If all the digits are zero,
                // replace with a single 0; otherwise, remove all
                // trailing zeros.
                String signif = Long.toHexString(signifBits).substring(3,16);
                answer.append(signif.equals("0000000000000") ? // 13 zeros
                              "0":
                              signif.replaceFirst("0{1,12}$", ""));

                answer.append('p');
                // If the value is subnormal, use the E_min exponent
                // value for double; otherwise, extract and report d's
                // exponent (the representation of a subnormal uses
                // E_min -1).
                answer.append(subnormal ?
                              DoubleConsts.MIN_EXPONENT:
                              Math.getExponent(d));
            }
            return answer.toString();
        }
    }


    public static Double valueOf(String s) throws NumberFormatException {
        return new Double(parseDouble(s));
    }

    /**
     * 返回表示指定的 double 值的 Double 实例。如果不需要新的 Double 实例，则通常应优先使用此方法，而不是
     * 构造方法 Double(double)，因为此方法可能通过缓存经常请求的值来显著提高空间和时间性能。
     */
    public static Double valueOf(double d) {
        return new Double(d);
    }

    /**
     * 返回一个新的 double 值，该值被初始化为用指定 String 表示的值，这与 Double 类的 valueOf 方法一样。
     */
    public static double parseDouble(String s) throws NumberFormatException {
        return FloatingDecimal.parseDouble(s);
    }

    /**
     * 如果指定的数是一个 NaN 值，则返回 true；否则返回 false。
     */
    public static boolean isNaN(double v) {
        return (v != v);
    }

    /**
     * 如果指定数在数值上为无穷大，则返回 true；否则返回 false。
     */
    public static boolean isInfinite(double v) {
        return (v == POSITIVE_INFINITY) || (v == NEGATIVE_INFINITY);
    }

    /**
     * 如果指定数在数值上不是无穷大，则返回 true；否则返回 false。
     */
    public static boolean isFinite(double d) {
        return Math.abs(d) <= DoubleConsts.MAX_VALUE;
    }

 	/**
     * 根据 IEEE 754 浮点双精度格式 ("double format") 位布局，返回指定浮点值的表示形式。
	第 63 位（掩码 0x8000000000000000L 选定的位）表示浮点数的符号。第 62-52 位（掩码 0x7ff0000000000000L 选定的位）表示指数。第 51-0 位（掩码 0x000fffffffffffffL 选定的位）表示浮点数的有效数字（有时也称为尾数）。
	如果参数是正无穷大，则结果为 0x7ff0000000000000L。
	如果参数是负无穷大，则结果为 0xfff0000000000000L。
	如果参数是 NaN，则结果为 0x7ff8000000000000L。
	在所有情况下，结果都是一个 long 整数，将其赋予 longBitsToDouble(long) 方法将生成一个与 doubleToLongBits 的参数相同的浮点值（所有 NaN 值被压缩成一个“规范”NaN 值时除外）。
     */
    public static long doubleToLongBits(double value) {
        long result = doubleToRawLongBits(value);
        // Check for NaN based on values of bit fields, maximum
        // exponent and nonzero significand.
        if ( ((result & DoubleConsts.EXP_BIT_MASK) ==
              DoubleConsts.EXP_BIT_MASK) &&
             (result & DoubleConsts.SIGNIF_BIT_MASK) != 0L)
            result = 0x7ff8000000000000L;
        return result;
    }
	
	// +
	public static double sum(double a, double b) {
        return a + b;
    }

    // max
    public static double max(double a, double b) {
        return Math.max(a, b);
    }

	// min
    public static double min(double a, double b) {
        return Math.min(a, b);
    }
```

#### 转换方法

```java
	/**
     * 以 byte 形式返回此 Double 的值（通过强制转换为 byte）
     */
    public byte byteValue() {
        return (byte)value;
    }

    /**
     * 以 short 形式返回此 Double 的值（通过强制转换为 short）
     */
    public short shortValue() {
        return (short)value;
    }

    /**
     * 以 int 形式返回此 Double 的值（通过强制转换为 int 类型）。
     */
    public int intValue() {
        return (int)value;
    }

    /**
     * 以 long 形式返回此 Double 的值（通过强制转换为 long 类型）
     */
    public long longValue() {
        return (long)value;
    }

    /**
     * 返回此 Double 对象的 float 值。
     */
    public float floatValue() {
        return (float)value;
    }

    /**
     * 返回此 Double 对象的 double 值。
     */
    public double doubleValue() {
        return value;
    }
```

#### 本地方法

```java
    /**
     * 根据 IEEE 754 浮点“双精度格式”位布局，返回指定浮点值的表示形式，并保留 NaN 值。
	第 63 位（掩码 0x8000000000000000L 选定的位）表示浮点数的符号。第 62-52 位（掩码 0x7ff0000000000000L 选定的位）表示指数。第 51-0 位（掩码 0x000fffffffffffffL 选定的位）表示浮点数的有效数字（有时也称为尾数）。
	如果参数是正无穷大，则结果为 0x7ff0000000000000L。
	如果参数是负无穷大，则结果为 0xfff0000000000000L。
	如果参数是 NaN，则结果是表示实际 NaN 值的 long 整数。与 doubleToLongBits 方法不同，doubleToRawLongBits 并没有压缩那些将 NaN 编码为一个“规范的”NaN 值的所有位模式。
	在所有情况下，结果都是一个 long 整数，将其赋予 longBitsToDouble(long) 方法将生成一个与 doubleToRawLongBits 的参数相同的浮点值。
     */
    public static native long doubleToRawLongBits(double value);

    /**
     * 返回对应于给定位表示形式的 double 值。根据 IEEE 754 浮点“双精度格式”位布局，参数被视为浮点值表示形式。
	如果参数是 0x7ff0000000000000L，则结果为正无穷大。
	如果参数是 0xfff0000000000000L，则结果为负无穷大。
	如果参数值在 0x7ff0000000000001L 到 0x7fffffffffffffffL 之间或者在 0xfff0000000000001L 到 0xffffffffffffffffL 之间，则结果为 NaN。Java 提供的任何 IEEE 754 浮点操作都不能区分具有不同位模式的两个同类型 NaN 值。不同的 NaN 值只能使用 Double.doubleToRawLongBits 方法区分。

在所有其他情况下，设 s、e 和 m 为可以通过以下参数计算的三个值：
 int s = ((bits >> 63) == 0) ? 1 : -1;
 int e = (int)((bits >> 52) & 0x7ffL);
 long m = (e == 0) ? (bits & 0xfffffffffffffL) << 1 :(bits & 0xfffffffffffffL) | 0x10000000000000L;
 
那么浮点结果等于算术表达式 s·m·2e-1075 的值。

注意，此方法不能返回与 long 参数具有完全相同位模式的 double NaN。IEEE 754 区分了两种 NaN：quiet NaN 和 signaling NaN。这两种 NaN 之间的差别在 Java 中通常是不可见的。对 signaling NaN 进行的算术运算将它们转换为具有不同（但通常类似）位模式的 quiet NaN。但是在某些处理器上，只复制 signaling NaN 也执行这种转换。特别是在复制 signaling NaN 以将其返回给调用方法时，可能会执行这种转换。因此，longBitsToDouble 可能无法返回具有 signaling NaN 位模式的 double 值。所以，对于某些 long 值，doubleToRawLongBits(longBitsToDouble(start)) 可能不 等于 start。此外，尽管所有 NaN 位模式（不管是 quiet NaN 还是 signaling NaN）都必须在上面提到的 NaN 范围内，但表示 signaling NaN 的特定位模式与平台有关。
     */
    public static native double longBitsToDouble(long bits);
```

#### hashCode与equals

```java
    /**
     *返回此 Double 对象的哈希码。结果是此 Double 对象所表示的基本 double 值的 long 整数位表示形式（与 doubleToLongBits(double) 方法生成的结果完全一样）两部分整数之间的异或 (XOR)。也就是说，哈希码就是以下表达式的值：(int)(v^(v>>>32))
	其中 v 的定义为：long v = Double.doubleToLongBits(this.doubleValue());
     */
	public static int hashCode(double value) {
        long bits = doubleToLongBits(value);
        return (int)(bits ^ (bits >>> 32));
    }

	/**
	 * 将此对象与指定对象比较。当且仅当参数不是 null 而是 Double 对象，且表示的 Double 值与此对象表示的 double 值相同时，结果为 true。为此，当且仅当将方法 doubleToLongBits(double) 应用于两个值所返回的 long 值相同时，才认为这两个 double 值相同。
	 如果 d1 和 d2 都表示 Double.NaN，那么即使 Double.NaN==Double.NaN 值为 false，equals 方法也将返回 true。
	如果 d1 表示 +0.0 而 d2 表示 -0.0，或者相反，那么即使 +0.0==-0.0 值为 true，equals 测试也将返回 false。
	 */
    public boolean equals(Object obj) {
        return (obj instanceof Double)
               && (doubleToLongBits(((Double)obj).value) ==
                      doubleToLongBits(value));
    }
```