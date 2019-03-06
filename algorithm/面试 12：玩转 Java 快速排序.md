## 面试 12：玩转 Java 快速排序

终于轮到我们排序算法中的王牌登场了。

快速排序由于排序效率在同为 O(nlogn) 的几种排序方法中效率最高，因此经常被采用。再加上快速排序思想——分治法也确实非常实用，所以 **在各大厂的面试习题中，快排总是最耀眼的那个**。要是你会的排序算法中没有快速排序，我想你还是偷偷去学好它，再去向大厂砸简历。

事实上，在我们的诸多高级语言中，都能找到它的某种实现版本，那我们 Java 自然不能在此缺席。

总的来说，默写排序代码是南尘非常不推荐的，撇开快排的代码不是那么容易默写，即使你能默写快排代码，也总会因为面试官稍微的变种面试导致你惶恐不安。

所以我们的面试系列自然不能少了这位王牌选手。

![图片来自于维基百科](https://upload.wikimedia.org/wikipedia/commons/6/6a/Sorting_quicksort_anim.gif)

## 基本思想

快速排序使用分治法策略来把一个序列分为两个子序列，基本步骤为：

1. 先从序列中取出一个数作为基准数；
2. 分区过程：将把这个数大的数全部放到它的右边，小于或者等于它的数全放到它的左边；
3. 递归地对左右子序列进行不走2，直到各区间只有一个数。

![图片来自于网络](https://upload.wikimedia.org/wikipedia/commons/thumb/8/84/Partition_example.svg/200px-Partition_example.svg.png)

虽然快排算法的策略是分治法，但分治法这三个字显然无法很好的概括快排的全部不走，因此借用 CSDN 神人 MoreWindows 的定义说明为：**挖坑填数 + 分治法**。

似乎还是不太好理解，我们这里就直接借用 MoreWindows 大佬的例子说明。

以一个数组作为示例，取区间第一个数为基准数。

| 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 72   | 6    | 57   | 88   | 60   | 42   | 83   | 73   | 48   | 85   |

初始时，i = 0;  j = 9;   temp = a[i] = 72

由于已经将 a[0] 中的数保存到 temp 中，可以理解成在数组 a[0] 上挖了个坑，可以将其它数据填充到这来。

从 j 开始向前找一个比 temp 小或等于 temp 的数。当 j = 8，符合条件，将 a[8] 挖出再填到上一个坑 a[0] 中。

a[0] = a[8]; i++;  这样一个坑 a[0] 就被搞定了，但又形成了一个新坑 a[8]，这怎么办了？简单，再找数字来填 a[8] 这个坑。这次从i开始向后找一个大于 temp 的数，当 i = 3，符合条件，将 a[3] 挖出再填到上一个坑中 a[8] = a[3]; j--;

数组变为：

| 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 48   | 6    | 57   | 88   | 60   | 42   | 83   | 73   | 88   | 85   |

 i = 3;   j = 7;   temp = 72

再重复上面的步骤，**先从后向前找，再从前向后找**。

从 j 开始向前找，当 j = 5，符合条件，将 a[5] 挖出填到上一个坑中，a[3] = a[5]; i++;

从i开始向后找，当 i = 5 时，由于 i==j 退出。

此时，i = j = 5，而a[5]刚好又是上次挖的坑，因此将 temp 填入 a[5]。

数组变为：

| 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 48   | 6    | 57   | 42   | 60   | 72   | 83   | 73   | 88   | 85   |

可以**看出 a[5] 前面的数字都小于它，a[5] 后面的数字都大于它**。因此再对 a[0…4] 和 a[6…9] 这二个子区间**重复**上述步骤就可以了。

对挖坑填数进行总结

1．i = L; j = R; 将基准数挖出形成第一个坑 a[i]。

2．j-- 由后向前找比它小的数，找到后挖出此数填前一个坑 a[i] 中。

3．i++ 由前向后找比它大的数，找到后也挖出此数填到前一个坑 a[j] 中。

4．再重复执行 2，3 二步，直到 i==j，将基准数填入 a[i] 中。

有了这样的分析，我们明显能写出下面的代码：

```java
public class Test09 {

    private static void printArr(int[] arr) {
        for (int anArr : arr) {
            System.out.print(anArr + " ");
        }
    }

    private static int partition(int[] arr, int left, int right) {
        int temp = arr[left];
        while (right > left) {
            // 先判断基准数和后面的数依次比较
            while (temp <= arr[right] && left < right) {
                --right;
            }
            // 当基准数大于了 arr[right]，则填坑
            if (left < right) {
                arr[left] = arr[right];
                ++left;
            }
            // 现在是 arr[right] 需要填坑了
            while (temp >= arr[left] && left < right) {
                ++left;
            }
            if (left < right) {
                arr[right] = arr[left];
                --right;
            }
        }
        arr[left] = temp;
        return left;
    }

    private static void quickSort(int[] arr, int left, int right) {
        if (arr == null || left >= right || arr.length <= 1)
            return;
        int mid = partition(arr, left, right);
        quickSort(arr, left, mid);
        quickSort(arr, mid + 1, right);
    }


    public static void main(String[] args) {
        int[] arr = {6, 4, 3, 2, 7, 9, 1, 8, 5};
        quickSort(arr, 0, arr.length - 1);
        printArr(arr);
    }
}
```

 我们不妨尝试来对这个算法进行一下时间复杂度的分析：

- 最好情况

  在最好的情况下，每次我们进行一次分区，我们会把一个序列刚好分为几近相等的两个子序列，这个情况也我们每次递归调用的是时候也就刚好处理一半大小的子序列。这看起来其实就是一个完全二叉树，树的深度为 O(logn)，所以我们需要做 O(logn) 次嵌套调用。但是在同一层次结构的两个程序调用中，不会处理为原来数列的相同部分。因此，程序调用的每一层次结构总共全部需要 O(n) 的时间。所以这个算法在最好情况下的时间复杂度为 O(nlogn)。

  事实上，我们并不需要如此精确的分区：即使我们每个基准值把元素分开为 99% 在一边和 1% 在另一边。调用的深度仍然限制在 100logn，所以全部运行时间依然是 O(nlogn)。

- 最坏情况

  事实上，我们总不能保证上面的理想情况。试想一下，假设每次分区后都出现子序列的长度一个为 1 一个为 n-1，那真是糟糕透顶。这一定会导致我们的表达式变成：

  T(n) = O(n) + T(1) + T(n-1) = O(n) + T(n-1)

  这和插入排序和选择排序的关系式真是如出一辙，所以我们的最坏情况是 O(n²)。

## 找到更好的基准数

上面对时间复杂度进行了简要分析，可见我们的时间复杂度和我们的基准数的选择密不可分。基准数选好了，把序列每次都能分为几近相等的两份，我们的快排就跟着吃香喝辣；但一旦选择的基准数很差，那我们的快排也就跟着穷困潦倒。

所以大家就各显神通，出现了各种选择基准数的方式。

- 固定基准数

  上面的那种算法，就是一种固定基准数的方式。如果输入的序列是随机的，处理时间还相对比较能接受。但如果数组已经有序，用上面的方式显然非常不好，因为每次划分都只能使待排序序列长度减一。这真是糟糕透了，快排沦为冒泡排序，时间复杂度为 O(n²)。因此，使用第一个元素作为基准数是非常糟糕的，我们应该立即放弃这种想法。

- 随机基准数

  这是一种相对安全的策略。由于基准数的位置是随机的，那么产生的分割也不会总是出现劣质的分割。但在数组所有数字完全相等的时候，仍然会是最坏情况。实际上，随机化快速排序得到理论最坏情况的可能性仅为1/(2^n）。所以随机化快速排序可以对于绝大多数输入数据达到 O(nlogn) 的期望时间复杂度。

- 三数取中

  虽然随机基准数方法选取方式减少了出现不好分割的几率，但是最坏情况下还是 O(n²)。为了缓解这个尴尬的气氛，就引入了「三数取中」这样的基准数选取方式。

## 三数取中法实现

我们不妨来分析一下「三数取中」这个方式。我们最佳的划分是将待排序的序列氛围等长的子序列，最佳的状态我们可以使用序列中间的值，也就是第 n/2 个数。可是，这很难算出来，并且会明显减慢快速排序的速度。这样的中值的估计可以通过随机选取三个元素并用它们的中值作为基准元而得到。事实上，随机性并没有多大的帮助，因此一般的做法是使用左端、右端和中心位置上的三个元素的中值作为基准元。显然使用三数中值分割法消除了预排序输入的不好情形，并且减少快排大约 5% 的比较次数。

我们来看看代码是怎么实现的。

```java
public class Test09 {

    private static void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }

    private static void printArr(int[] arr) {
        for (int anArr : arr) {
            System.out.print(anArr + " ");
        }
    }

    private static int partition(int[] arr, int left, int right) {
        // 采用三数中值分割法
        int mid = left + (right - left) / 2;
        // 保证左端较小
        if (arr[left] > arr[right])
            swap(arr, left, right);
        // 保证中间较小
        if (arr[mid] > arr[right])
            swap(arr, mid, right);
        // 保证中间最小，左右最大
        if (arr[mid] > arr[left])
            swap(arr, left, mid);
        int pivot = arr[left];
        while (right > left) {
            // 先判断基准数和后面的数依次比较
            while (pivot <= arr[right] && left < right) {
                --right;
            }
            // 当基准数大于了 arr[right]，则填坑
            if (left < right) {
                arr[left] = arr[right];
                ++left;
            }
            // 现在是 arr[right] 需要填坑了
            while (pivot >= arr[left] && left < right) {
                ++left;
            }
            if (left < right) {
                arr[right] = arr[left];
                --right;
            }
        }
        arr[left] = pivot;
        return left;
    }

    private static void quickSort(int[] arr, int left, int right) {
        if (arr == null || left >= right || arr.length <= 1)
            return;
        int mid = partition(arr, left, right);
        quickSort(arr, left, mid);
        quickSort(arr, mid + 1, right);
    }


    public static void main(String[] args) {
        int[] arr = {6, 4, 3, 2, 7, 9, 1, 8, 5};
        quickSort(arr, 0, arr.length - 1);
        printArr(arr);
    }
}
```

由于篇幅关系，今天我们的讲解暂且就到这里。

话说 Java 官方是怎么实现的呢？我们明天不妨直接到 JDK 里面一探究竟。