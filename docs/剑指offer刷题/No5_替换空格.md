# No5 替换空格

## 题目

请实现一个函数，把字符串 s 中的每个空格替换成"%20"。

 

> 示例 1：
>
> 输入：s = "We are happy."
> 输出："We%20are%20happy."


限制：

0 <= s 的长度 <= 10000

## 分析

一个个遍历然后替换，可能需要注意的是单线程下StringBuilder比StringBuffer快吧。

## 解法

#### 解法1 遍历

```java
class Solution {
    public String replaceSpace(String s) {
        StringBuilder sb = new StringBuilder();
        for (char c : s.toCharArray()){
            if (c == ' '){
                sb.append("%20");
            } else {
                sb.append(c);
            }
        }
        return sb.toString();
    }
}
```

#### 解法2 库函数

```java
class Solution {
    public String replaceSpace(String s) {
    	if(s == null || s.length() == 0) return s;
  		return s.replaceAll(" ", "%20");
    }
}
```

