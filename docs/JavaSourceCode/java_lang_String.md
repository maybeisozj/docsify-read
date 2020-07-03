# java.lang.String 阅读总结

## 概要

String 是一些字符(Character，**UTF-16**)的集合，例如"abc"。String 是final修饰的，因此它是不可修改的，是一个常量。Java语言为字符串连接运算符（**+**）以及将其他对象转换为字符串提供了特殊的支持。字符串连接是通过`StringBuilder`（或`StringBuffer`）类及其`append`方法实现的。平时我们如果需要使用不断修改(**添加与删除字符**)推荐使用StringBuffer与StringBuilder类，前者是线程不安全的后者反之。

String 类提供了： 检查序列中的各个字符，比较字符串，搜索字符串，提取子字符串以及创建字符串的副本，并将所有字符转换为大写或小写。大小写映射基于[`Character`](https://docs.oracle.com/javase/7/docs/api/java/lang/Character.html)该类指定的Unicode标准版本。

它有以下属性：

- final char[] value //用于存储字符串
- int hash // 存储字符串的hash值
- static final ObjectStreamField[] serialPeristentFields //ObjectStreamFields数组用于声明类的Serializable字段

## 方法

### 构造方法

#### String()

创建一个String对象代表空字符串。

#### String (String original)

创建original的一个copy，对新产生的String对象的修改不会影响到original。**除非需要显式的创建一个original的副本，该方法并无大用，因为String是无法修改的。**

#### String （char[] value）

创建一个String对象表示字符数组参数中当前包含的字符序列。**对String对象的修改不会影响value数组。**

#### String (char[] value, int offest, int count)

创建一个String对象表示字符数组参数中从offset开始的count个字符序列。

#### String(int[] codePoints, int offset, int count)

分配一个新`String`字符，其中包含来自[Unicode代码点数](https://docs.oracle.com/javase/7/docs/api/java/lang/Character.html#unicode)组参数的子数组的[字符](https://docs.oracle.com/javase/7/docs/api/java/lang/Character.html#unicode)。该`offset`参数是子阵列的第一码点的指数和`count`参数指定子阵列的长度。子数组的内容转换为 `char`s；`int`数组的后续修改不会影响新创建的字符串。

查看源码可以知道该方法流程为：

1. 首先计算代码点代表的字符个数(**中间会穿插代码点的检验**)，
2. 然后分配char[] 空间，
3. 最后把代码点转为char并赋值给value数组。

#### String(byte ascii[], int hibyte, int offset, int count)

**不推荐使用。** *此方法不能正确将字节转换为字符。从JDK 1.1开始，执行此操作的首选方法是通过`String`采用[`Charset`](https://docs.oracle.com/javase/7/docs/api/java/nio/charset/Charset.html)，字符集名称或使用平台默认字符集的 构造函数。*

`String`从8位整数值数组的子数组中分配一个新构造的对象。

该`offset`参数是子阵列的第一个字节的索引，和`count`参数指定子阵列的长度。

`byte`子数组中的每个都将转换为`char`上述方法中指定的。

#### String(byte ascii[], int hibyte)

同上。

#### String(byte bytes[], int offset, int length, String charsetName)

`String`通过使用指定的字符集对指定的字节子数组进行解码来构造一个String对象 。新建的String的长度是字符集的函数，因此可能不等于子数组的长度。

另外还有许多调用该方法的构造方法，不一一列举了。

#### String(StringBuffer buffer)

分配一个新字符串，该字符串包含当前在 buffer 参数中包含的字符序列。buffer 的内容被复制；同时 buffer 的后续修改不会影响新创建的字符串。

#### String(StringBuilder builder)

分配一个新字符串，该字符串包含当前在 builder 参数中包含的字符序列。builder 的内容被复制；同时 builder 的后续修改不会影响新创建的字符串。**通过`toString`方法从 builder 中获取字符串可能会运行得更快，通常是首选。**

### 其他方法

#### int length()

返回String的长度，即**value.length**。

#### boolean isEmpty()

如果String为空则返回true，反之返回false。

#### char charAt(int index)

返回指定下标的位置的字符，如果参数index不在value的范围内抛出**StringIndexOutOfBoundsException**异常。

此方法可以通过下标随机访问是因为String的字符是用**数组**存储的，同时，我们也可以使用toCharArray()方法获取到value数组然后进行随机访问。

#### int codePointAt（int index）

返回指定索引处的字符（Unicode代码点）。索引引用`char`值（Unicode代码单位），范围从`0`到 length() - 1

如果`char`在给定索引处指定的值在高代理范围内，则后续索引小于此值的长度`String`，而 `char`在后续索引处的值在低代理范围内，则与此对应的补充代码点返回代理对。否则，将`char`返回给定索引处的值。

#### void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin)

将字符串中的字符复制到目标字符数组中。

要复制的第一个字符在索引处`srcBegin`; 最后要复制的字符在索引处`srcEnd-1` （因此要复制的字符总数为 `srcEnd-srcBegin`）。字符被复制到`dst`从索引开始到索引`dstBegin` 结束的子数组中：
$$
dstbegin +（srcEnd-srcBegin）-1
$$
**无论是超出 value 数组范围还是 dst 数组范围都会抛出 IndexOutOfBoundsException**

#### byte[] getBytes(Charset  charset)

`String`使用给定的字符集将其编码为字节序列 ，并将结果存储到新的字节数组中。

此方法始终使用此字符集的默认替换字节数组替换格式错误的输入和不可映射的字符序列。在 [`CharsetEncoder`](https://docs.oracle.com/javase/7/docs/api/java/nio/charset/CharsetEncoder.html)需要在编码处理更多的控制类应当被使用。

#### boolean equals(Object anObject)

String重写了Object的equals方法，并不是检验地址是否相同而是value数组里的值是否相同。

流程：

1. 转换anObject为String
2. 判断长度是否相同
3. 依次判断字符是否相同

#### boolean contentEquals(StringBuffer  sb)

将此字符串与指定的比较`StringBuffer`。结果是，`true`当且仅当它`String`表示与指定相同的字符序列`StringBuffer`。使用contentEquals(CharSequence  cs)方法。

#### boolean contentEquals(CharSequence  cs)

将此字符串与指定的比较`CharSequence`。结果是，`true`当且仅当它`String`表示与指定序列相同的char值序列。

#### boolean equalsIgnoreCase(String  anotherString)

将其`String`与另一个比较`String`，忽略大小写考虑。如果两个字符串的长度相同，并且两个字符串中的对应字符忽略大小写是相等返回true。

#### int compareTo( String  anotherString)

按字典顺序比较两个字符串。比较是基于字符串中每个字符的Unicode值。在`String`字典上比较**此对象**表示 的字符序列与**anotherString**表示的字符序列。**如果此`String`对象在字典上在anotherString之前，则结果为负整数。如果此`String`对象在字典上位于anotherString后面，则结果为正整数。如果字符串相等，则结果为零；否则，结果为零。`compareTo`返回方法`0`时[`equals(Object)`](https://docs.oracle.com/javase/7/docs/api/java/lang/String.html#equals(java.lang.Object))返回`true`**。

> 字典顺序的定义是：
>
> 如果两个字符串不同，那么它们要么在某个索引（**有效索引**）处具有不同的字符，要么它们的长度不同，或者两者都存在。
>
> 如果它们在一个或多个索引位置具有不同的字符，则令*k*为最小索引；那么，在字符串位置*k*处的字符具有较小值（通过使用<运算符确定）的字符串，按字典顺序在另一个字符串之前。在这种情况下，`compareTo`返回`k`两个字符串位置处两个字符值的差值，即值：
> $$
> this.charAt（k）-anotherString.charAt（k）
> $$
> 如果在它们之间没有索引位置不同，则较短的字符串在字典上在较长的字符串之前。在这种情况下， `compareTo`返回字符串长度的差值，即值：
> $$
>  this.length（）-anotherString.length（）
> $$
> 

#### int compareToIgnoreCase( String  anotherString)

忽略字符大小写差异的情况下按字典顺序比较两个字符串。

#### boolean startsWith(String prefix, int toffset)

测试此字符串的子字符串是否**从指定的索引开始**是以**指定的前缀**开头的。

#### boolean startsWith(String prefix)

测试此字符串是否是以**指定的前缀**开头的。调用startsWith(prefix, 0)。

#### boolean endsWith(String suffix)

测试此字符串是否是以**指定的后缀**结尾的。调用**stratsWith(suffix, value.length - suffix.value.length)**。

#### int  hashCode()

返回String 对象的hash值。

返回此字符串的哈希码。String对象的哈希码计算为

$$
s[0] * 31^(n-1) + s [1] * 31 ^(n-2) + ... + s [n-1]
$$
使用int算术，其中s[i]是 我字符串的个字符，n是字符串的长度，以及^表示取幂。（空字符串的哈希值为零。）

```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

#### int indexOf(int ch, int fromIndex)

返回第一次出现的指定字符在此字符串内的索引，从指定索引开始搜索。

如果具有值`ch`的字符出现在此`String` 对象表示的字符序列中的索引不小于`fromIndex`，则返回第一次出现的索引。

源码实现就是从fromIndex开始遍历查找，相同即返回，直到最后仍然没有找到就返回-1。

#### int lastIndexOf(int ch, int fromIndex)

返回最后一次出现的指定字符在此字符串内的索引。

源码实现就是从后往前遍历，找到即返回，没有找到就返回-1.

#### String substring(int beginIndex, int endIndex)

返回一个新字符串，该字符串是该字符串的子字符串。子字符串从指定的字符串`beginIndex`开始并扩展到索引endIndex - 1`处的字符。因此，子字符串的长度为`endIndex-beginIndex`。**不包含endIndex处的字符。**

#### String concat(String str)

如果参数字符串的长度为`0`，则`String`返回此 对象。

否则，将`String`创建一个新对象，该 对象表示一个字符序列，该字符序列是此`String`对象表示的字符序列和自变量字符串表示的字符序列的串联。直接创建新数组，通过Arrays.copy()拷贝value数组，通过str.getChars() 方法拷贝str对象。

另外一种链接"+"是通过StringBuffer或者StringBuilder实现的。

#### String replace(char oldChar, char newChar)

返回一个新字符串，该字符串是用`newChar`替换该字符串中所有出现的oldChar的新字符串。

#### boolean contains(CharSequence s)

查看该字符串是否包含s。

#### String replaceFirst(String regex, String replacement)

用给定的replacement替换与给定的[正则表达式](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html#sum) regex 匹配的此字符串的第一个子字符串。

#### String replaceAll(String regex, String replacement)

用给定的replacement替换与给定的[正则表达式](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html#sum) regex 匹配的此字符串的所有子字符串。

#### String[] split(String regex, int limit)

在给定[正则表达式的](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html#sum)匹配项周围拆分此字符串 。

此方法返回的数组包含此字符串的每个子字符串，该子字符串由与给定表达式匹配的另一个子字符串终止或由字符串末尾终止。数组中的子字符串按照它们在此字符串中出现的顺序排列。如果表达式与输入的任何部分都不匹配，则结果数组只有一个元素，即此字符串。

的`极限`参数控制的被施加图案的次数，并因此影响所得阵列的长度。如果限制*n*大于零，则将最多应用*n* -1次该模式，该数组的长度将不大于*n*，并且该数组的最后一个条目将包含除最后一个匹配的定界符之外的所有输入。如果*n* 为非正数，则将尽可能多地应用该模式，并且数组可以具有任何长度。如果*n*为零，则该模式将被尽可能多地应用，该数组可以具有任意长度，并且尾随的空字符串将被丢弃。

#### String[] spilt(String regex)

调用是spilt(regex, 0)返回结果。

#### String toLowerCase(Locale locale)

`String`使用给定的规则将所有字符转换为小写`Locale`。

#### String toLowerCase()

String使用默认语言环境的规则将此字符转换为小写。这相当于调用 toLowerCase(Locale.getDefault())。

#### String toUpperCase(Locale locale)

`String`使用给定的规则将所有字符转换为大写`Locale`。

#### String toUpperCase()

String使用默认语言环境的规则将此字符转换为大写。这相当于调用 toUpperCase(Locale.getDefault())。

#### String trim()

返回字符串的副本，省略前导和尾随空格。

如果此`String`对象表示一个空字符序列，或者此对象表示的字符序列的第一个和最后一个字符`String`都具有大于`'\u0020'`（空格字符）的代码，则`String`返回对该对象的引用。

#### char[] toCharArray()

将此字符串转换为新的字符数组。