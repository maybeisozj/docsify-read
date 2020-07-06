# No3 数组中重复的数字

## 题目

找出数组中重复的数字。


在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

>  示例 1：
>
> 输入：
> [2, 3, 1, 0, 2, 5, 3]
> 输出：2 或 3 


限制：

2 <= n <= 100000

## 分析

看到题目的第一眼想到的就是**Set**了，因为它天生就是无法添加重复元素的，如果添加失败那么这个元素就是已经出现过了，所以他就是重复的元素。不过使用Set的话，我们需要一些额外的空间，最大需要n-1，而且每次插入新元素的时候我们需要求出hashcode，进行equals比较。

使用Set的原因是它的唯一性，那么我们使用数组来实现哈希表就行了，不需要使用Set。注意题目中的**数字都在 0～n-1 的范围内**，这正好与数组下标相符合，那么我们新建一个长度为n 的数组，遍历原数组把每个数放置到对应的下标处，即0放置到下标为0的地方，如果放置时下标为i的元素已经是i了，那么这个数字就是重复出现了的。这种解法虽然依旧使用了额外的空间，但是插入的时候操作没有那么多了。

那么我们可以在原数组上操作嘛？因为这样的话不需要占用新的空间。依据我们上面的思路的话，我们是根据下标是否与元素对应来进行判断的，那么我们只要把数字放到它该放的地方去就好了(如数字1放置到下标1的地方去)，在放置的过程中如果出现了下标为i的元素值已经是i了，那么这个数字就是重复出现的数字了。

还有一种就是不利用下标与元素对应的特性，而是将数组进行排序，然后遍历的时候如果临近的两个元素相等那么就是重复出现的数字了。

## 解法

### 解法1 利用Set的特性

使用set的特性，添加失败就是已经存在了即重复的数字。

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        Set<Integer> set = new HashSet<Integer>();
        int ans = 0;
        for (int i : nums){
            if (!set.add(i)){
                ans = i;
                break;
            }
        }
        return ans;
    }
}
```

### 解法2 哈希表

新建数组实现哈希表，如果数量大于2就是重复出现了。

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        int[] t = new int[nums.length];
        int ans = 0;
        for (int i : nums){
            t[i] ++;
            if (t[i] >= 2){
                ans = i;
                break;
            }
        }
        return ans;
    }
}
```

### 解法3 原地哈希

原地修改数组使得数字i放置到下标i的位置，出现下标i的元素已经是i的时候就是找到了重复的元素了。

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        for (int i = 0;i < nums.length; i ++){
            while (i != nums[i]){
                if (nums[i] == nums[nums[i]]){
                    return nums[i];
                }
                int temp = nums[i];
                nums[i] = nums[temp];
                nums[temp] = temp;
            }
        }
        return nums[0];
    }
}
```

### 解法4 排序查找

排序之后两两比较如果相等的话就是重复出现的。

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        Arrays.sort(nums);
        int ans = 0;
        for (int i = 1;i < nums.length;i++){
            if (nums[i] == nums[i-1]){
                ans = nums[i];
                break;
            }
        }
        return ans;
    }
}
```

