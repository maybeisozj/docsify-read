# No4 二维数组中的查找

## 题目

在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

 

> 示例:
>
> 现有矩阵 matrix 如下：
>
> [
>   [1,   4,  7, 11, 15],
>   [2,   5,  8, 12, 19],
>   [3,   6,  9, 16, 22],
>   [10, 13, 14, 17, 24],
>   [18, 21, 23, 26, 30]
> ]
> 给定 target = 5，返回 true。
>
> 给定 target = 20，返回 false。

限制：

0 <= n <= 1000

0 <= m <= 1000

## 分析

暴力法，将整个数组遍历一次，如果最后没有找到的话就是不含该数字。

我们再仔细看看题目，每一行每一列都是一个递增的，那么我们就可以利用它的这个特性，使用二分法呀。那么怎么使用呢？单独对一行或者一列进行二分！如果只对一行或者一列进行二分是不是有点浪费呢？那么要如何同时使用好数组的特性呢？

讲真，个人我也就想到了上面那里的逐行或逐列进行二分。在评论区里我看见了一句话，[站在右上角看。这个矩阵其实就像是一个Binary Search Tree。然后，聪明的大家应该知道怎么做了。](https://leetcode-cn.com/u/ymy1248/)所以树嘛，不是左子树就是右子树，遍历完了没有结果就是没有出现过呀。

## 解法

### 解法1 暴力法

逐个进行遍历，找到了结果就返回true，最后没有就返回false。

```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        // 特殊处理
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0){
            return false;
        }
        int n = matrix.length;
        int m = matrix[0].length;
        for (int i = 0; i < n;i ++){
            for (int j = 0;j < m;j ++){
                // 逐个遍历查找
                if (matrix[i][j] == target){
                    return true;
                }
            }
        }
        // 最后还是没有找到就是不存在
        return false;
    }
}
```

### 解法2 逐行二分

对每一行进行二分查找，找到了就返回true，最后没有找到就返回false。当然你也可以逐列查找。

```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        // 特殊处理
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0){
            return false;
        }
        int n = matrix.length;
        int m = matrix[0].length;
        // 每一行进行二分
        for (int i = 0; i < n; i++) {
            int l = 0, r = m - 1;
            while (l <= r) {
                int mid = (l + r) / 2;
                if (target == matrix[i][mid]) {
                    return true;
                } else 
                    if (target < matrix[i][mid]){
                        r = mid - 1;
                    } else {
                        l = mid + 1;
                    }
            }
        }
        return false;
    }
}
```

### 解法3 树

从右上角开始将数组看做树，target大于当前值向下，小于向左。(**这里我们也可以从左下角开始，因为上面是比他小的，左边是比它大的，也是一棵二叉搜索树呢。**)

```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        // 特殊处理
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0){
            return false;
        }
        int n = matrix.length;
        int m = matrix[0].length;
        // 从右上角开始
        int col = 0, row = m - 1;
        while (col < n && row >= 0){
            // 小于当前值向左
            if (matrix[col][row] > target){
                row --;
            } 
            // 大于当前值向下
            else if (matrix[col][row] < target){
                col ++;
            } else {
                // 等于就是我们需要的结果
                return true;
            }
        }
        return false;
    }
}
```

