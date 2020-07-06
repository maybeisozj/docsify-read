# java.lang.Float 阅读总结

## 概要

Float是float基础类型的包装类，它继承了Number类并实现了Comparable接口，它还提供了与String类型转换的相关函数。

**他与Double的差别在于精度，体现在size(w位数)上。**

它有正无穷，负无穷，NaN(不是数)，最大值，最小值，最小正标准值的常量，最小正非零值的常量，具体如下：

```java
	/**
     * 正无穷
     */
    public static final float POSITIVE_INFINITY = 1.0f / 0.0f;

    /**
     * 负无穷
     */
    public static final float NEGATIVE_INFINITY = -1.0f / 0.0f;

    /**
     * NaN常量（Not a Number）
     */
    public static final float NaN = 0.0f / 0.0f;

    /**
     * 最大正有限值的常量
     * (2-2<sup>-32</sup>)&middot;2<sup>127</sup>.
     */
    public static final float MAX_VALUE = 0x1.fffffeP+127f; // 3.4028235e+38f

    /**
     * 最小正标准值的常量 2<sup>-126</sup>.  
     */
    public static final float MIN_NORMAL = 0x1.0p-126f; // 1.17549435E-38f

    /**
     * 最小正非零值的常量 2<sup>-149</sup>. 
     */
    public static final float MIN_VALUE = 0x0.000002P-126f; // 1.4e-45f

    /**
     * 可能具有的最大指数
     */
    public static final int MAX_EXPONENT = 127;

    /**
     * 可能具有的最小指数
     */
    public static final int MIN_EXPONENT = -126;

    /**
     * 位数
     */
    public static final int SIZE = 326;

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
    public static final Class<Float> TYPE = (Class<Float>) Class.getPrimitiveClass("float");
	
	/**
	 * 值
	 */
	private final float value;

	/**
	 * 序列化
 	 */
	private static final long serialVersionUID = -2671257302660747028L;
```



## 方法

见Double
