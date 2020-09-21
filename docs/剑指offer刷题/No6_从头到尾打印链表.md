# No6 从头到尾打印链表

## 题目

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

> 示例 1：
>
> 输入：head = [1,3,2]
> 输出：[2,3,1]


限制：

0 <= 链表长度 <= 10000

节点定义:

```java
    public class ListNode {
         int val;
         ListNode next;
         ListNode(int x) { val = x; }
    }
```

## 分析

逆序，说实话，我想到的是stack，先进后出嘛，而且也是单向链表嘛。

分析分析上面的stack，我们准确来说，是遍历了两次的，一次在数组，一次在栈，那么我遍历两次数组不就可以了嘛？为什么还要用栈呢？是吧~

然后，使用递归调用的，记得树的几种遍历嘛。改变递归调用的位置我们就可以实现先序，中序，后序的快速转换，那么在这里，我们先进行递归然后才添加元素就可以实现题目要求的逆序输出了。**具体见代码，这里感觉讲不清楚。**

其实如果用list的话，一遍遍历应该就可以了，因为我们可以从头开始加呀，嘿嘿。

## 解法

### 解法1 stack

利用stack存储所有的结果，然后依次输出到数组里即可。

```java
class Solution {
    public int[] reversePrint(ListNode head) {
        Stack<Integer> stack = new Stack<Integer>();
        while (head != null){
            stack.push(head.val);
            head = head.next;
        }
        int n = stack.size();
        int[] ans = new int[n];
        for (int i = 0;i < n;i ++){
            ans[i] = stack.pop();
        }
        return ans;
    }
}
```

### 解法2 两次遍历数组

第一次遍历取到链表的长度，然后第二次把链表的节点的值插入到对应的位置即可。

```java
class Solution {
    public static int[] reversePrint(ListNode head) {
        ListNode node = head;
        int count = 0;
        while (node != null) {
            ++count;
            node = node.next;
        }
        int[] nums = new int[count];
        node = head;
        for (int i = count - 1; i >= 0; --i) {
            nums[i] = node.val;
            node = node.next;
        }
        return nums;
    }
}
```

### 解法3 递归的使用

递归调用，首先添加的是最后一个，依次回调添加，最后用流转换为数组。

这里我给了两种写法，一种是使用了全局变量的，一种是具备变量。速度是1比2快，嘿嘿，因为2每次进行函数调用的时候需要传入list，栈桢有点大的。单线程不需要考虑安全问题，推荐写法1，实际工程上但是推荐写法2。

```java
// 写法1
class Solution {

    List<Integer> ans = new ArrayList<Integer>();

    public int[] reversePrint(ListNode head) {
        reversePrint1(head);
        return ans.stream().mapToInt(Integer::valueOf).toArray();
    }

    public void reversePrint1(ListNode head){
        if (head == null){
            return;
        }
        reversePrint1(head.next);
        ans.add(head.val);
    }
}

// 写法2
class Solution {

    public int[] reversePrint(ListNode head) {
        List<Integer> ans = new ArrayList<Integer>();
        reversePrint1(head,ans);
        return ans.stream().mapToInt(Integer::valueOf).toArray();
    }

    public void reversePrint1(ListNode head,List<Integer> ans){
        if (head == null){
            return;
        }
        reversePrint1(head.next,ans);
        ans.add(head.val);
    }
}
```

### 解法4 从头开始加

每次都是从头开始加的，这样就添加完成之后就是逆序的了。

```java
class Solution {

    public int[] reversePrint(ListNode head) {
        List<Integer> ans = new ArrayList<Integer>();
        while (head != null){
            ans.add(0,head.val);
            head = head.next;
        }
        return ans.stream().mapToInt(Integer::valueOf).toArray();
    }
}
```



