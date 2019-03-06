# 面试 19：输出数组中出现次数超过一半的数字（剑指 Offer 26 题）

上一篇推文给大家留下的习题来自于《剑指 Offer》第 29 题：数组中超过一半的数字，不知道各位去思考了么？

> 面试题：数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字并输出。比如 {1,2,3,2,2,2,1} 中 2 的次数是 4，数组长度为 7，所以输出 2。要求不能修改输入的数组。

## 准备测试用例

这道题能思考到的测试用例比较简单。

1. 输入符合条件的数组，查看打印是否满足情况；
2. 输入不符合条件的数组，查看打印；
3. 输入只有一个元素的数组，查看打印；
4. 输入无效数组，查看打印；

## 思考程序逻辑

第二步便是我们的思考程序逻辑了，题目要求查找出现次数超过一半的数字。比较容易想到的思路是直接对数组排序，那中间那个值就是我们想要的值，但这样的想法明显排序后会对输入的数组顺序有影响，所以我们可能需要换一种思路。

再看一遍题干，我们不难思考到，我们是否可以对每个数字进行计数，最后返回计数次数最多的值。存储次数采用 map 做映射处理。

```java
public class Test19 {
    private static int moreThanHalfNums(int[] nums) {
        if (nums == null || nums.length == 0)
            throw new RuntimeException("the length of array must be large than 0");
        int len = nums.length;
        Map<Integer, Integer> map = new HashMap<>();

        for (int num : nums) {
            if (map.containsKey(num))
                map.put(num, map.get(num) + 1);
            else
                map.put(num, 1);
        }
        int times = len / 2;
        // 查找 map 中 value 最大的值
        for (Entry<Integer, Integer> entry : map.entrySet()) {
            if (entry.getValue() > times)
                return entry.getKey();
        }
        throw new RuntimeException("invalid input!");
    }

    public static void main(String[] args) {
        int[] nums1 = {1, 2, 3, 2, 2, 4, 2, 2, 5};
        System.out.println(moreThanHalfNums(nums1));
        int[] nums2 = {1};
        System.out.println(moreThanHalfNums(nums2));
        int[] nums3 = {2, 1, 2, 1, 2, 2, 3, 2, 1};
        System.out.println(moreThanHalfNums(nums3));
        int[] nums4 = {1, 2, 3, 4, 5};
        System.out.println(moreThanHalfNums(nums4));
    }
}
```

写毕后进行测试用例的验证，无不例外，目前都通过，于是我们把这样的代码解法递交给面试官。

面试官看了这样的算法，表示他更期待的是不使用任何辅存空间的算法。于是我们得换个角度思考。

数组中有一个数字出现的次数超过数组长度的一半，也就是说它出现的次数比其他所有数字出现次数的和还要多。因此我们可以考虑在遍历数组的时候保存两个值： 一个是数组中的一个数字， 一个是次数。当我们遍历到下一个数字的时候，如果下一个数字和我们之前保存的数字相同，则次数加 1 ；如果下一个数字和我们之前保存的数不同，则次数减 1。如果次数为 0，我们需要保存下一个数字，并把次数设为 1 。由于我们要找的数字出现的次数比其他所有数字出现的次数之和还要多，那么要找的数字肯定是最后一次把次数设为 1 时对应的数字。

我们来看这样的思路用代码怎么实现。

```java
public class Test19 {

    private static int moreThanHalfNums(int[] nums) {
        if (nums == null || nums.length == 0)
            throw new RuntimeException("the length of array must be large than 0");
        int result = nums[0];
        int times = 1;
        int len = nums.length;
        for (int i = 1; i < len; i++) {
            if (times == 0) {
                result = nums[i];
                times = 1;
            } else if (result == nums[i])
                times++;
            else
                times--;
        }
        times = 0;
        for (int num : nums) {
            if (num == result)
                times++;
        }
        if (times > len / 2)
            return result;
        throw new RuntimeException("invalid input!");
    }

    public static void main(String[] args) {
        int[] nums1 = {1, 2, 3, 2, 2, 4, 2, 2, 5};
        System.out.println(moreThanHalfNums(nums1));
        int[] nums2 = {1};
        System.out.println(moreThanHalfNums(nums2));
        int[] nums3 = {2, 1, 2, 1, 2, 2, 3, 2, 1};
        System.out.println(moreThanHalfNums(nums3));
        int[] nums4 = {1, 2, 3, 4, 5};
        System.out.println(moreThanHalfNums(nums4));
    }
}
```

写毕后，验证测试用例，同样全部通过。 

本题最后的思路，希望大家刻意去思考和记忆一下，因为也许变一下题意，这样的想法还可以用到。

## 课后习题

我们下一次推文的题目来源于《剑指 Offer》第 31 题：计算连续子数组的最大和。

> 面试题：输入一个整型数组，数组中有正数也有负数。数组中一个或多个整数形成一个子数组，求所有子数组的和的最大值，要求时间复杂度为 O(n)。
> 比如输入 {1, -2, 3, 10, -4, 7, 2, -5}，能产生子数组最大和的子数组为 {3,10,-4,7,2}，最大和为 18。