## 面试 9：用 Java 实现冒泡排序

南尘的朋友们，新的一周好，原本打算继续讲链表考点算法的，这里姑且是卡一段。虽然在我们 Android 开发中，很少涉及到排序算法，因为基本官方都帮我们封装好了，但排序算法也是非常重要的，在面试中 **归并排序** 和 **快速排序** 一直为高频考点，但在学习它们之前，我们必须得先把三大基础算法学会，毕竟层层递进，方得始终嘛。

### 冒泡排序

冒泡排序恐怕是我们计算机专业课程上以第一个接触到的排序算法，也算是一种入门级的排序算法。它的基本思想是：**两两比较相邻记录的关键字，如何反序则交换，直到没有反序的记录为止。**

#### 冒泡排序算法原理：

1. 比较相邻的元素。如果第一个比第二个大，就交换他们两个。
2. 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。这步做完后，最后的元素会是最大的数。
3. 针对所有的元素重复以上的步骤，除了最后一个。
4. 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

一次比较过程如图所示：

![图片来源于网络](https://user-gold-cdn.xitu.io/2018/3/1/161e0ae4c75fd077?imageslim)

我们通常容易想到最简单的实现代码：

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

    private static void bubbleSort(int[] arr) {
        if (arr == null)
            return;
        for (int i = 0; i < arr.length - 1; i++) {
            for (int j = i + 1; j < arr.length; j++) {
                if (arr[i] > arr[j])
                    swap(arr, i, j);
            }
        }
    }

    public static void main(String[] args) {
        int[] arr = {6, 4, 2, 1, 8, 3, 7, 9, 5};
        bubbleSort(arr);
        printArr(arr);
    }
}
```

严格地讲，上面的算法并不是冒泡排序，因为 **它完全不符合两两相邻比较。**它更应该是最最简单的就交换排序而已。它的思路是让每一个关键字，都和它后面的每一个关键字比较，如果大则交换，这样第一位置的关键字在一次循环后一定变成最小值。

我们不妨来看看正宗的冒泡排序算法。

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

    private static void bubbleSort(int[] arr) {
        if (arr == null)
            return;
        for (int i = 0; i < arr.length - 1; i++) {
            for (int j = 1; j < arr.length - i; j++) {
                if (arr[j - 1] > arr[j]) {
                    swap(arr, j - 1, j);
                }
            }
        }
    }

    public static void main(String[] args) {
        int[] arr = {6, 4, 2, 1, 8, 3, 7, 9, 5};
        bubbleSort(arr);
        printArr(arr);
    }
}
```

上述代码是否完美了呢？答案是否定的，我们假设待排序的序列是 {2，1，3，4，5，6，7，8，9}，也就是说，除了第一和第二个关键字需要交换外，别的都应该是正常的顺序，当 i = 1 时，交换了 2 和 1 的位置，此时已经有序，但是算法依然不依不挠地将 i = 2 到 9 以及每一个内循环都执行了一遍，尽管没有交换数据，但之后的大量比较还是大大的多余了。所以我们完全可以设置一个标记位 `isSort`，当我们比较一次后都没有交换，则代表数组已经有序了，此时直接退出循环即可。

既然思路已经确定，那代码自然是很信手拈来了。

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

    private static void bubbleSort(int[] arr) {
        if (arr == null)
            return;
        // 定义一个标记 isSort，当其值为 true 的时候代表已经有序。
        boolean isSort;
        for (int i = 0; i < arr.length - 1; i++) {
            isSort = true;
            for (int j = 1; j < arr.length - i; j++) {
                if (arr[j - 1] > arr[j]) {
                    swap(arr, j - 1, j);
                    isSort = false;
                }
            }
            if (isSort)
                break;
        }
    }

    public static void main(String[] args) {
        int[] arr = {6, 4, 2, 1, 8, 3, 7, 9, 5};
        bubbleSort(arr);
        printArr(arr);
    }
}
```

Perfect 的代码，但冒泡排序在数组长度较大的时候，效率真的很低下，所以在实际生产中，我们也很少使用这种算法。

#### 冒泡排序时间空间复杂度及算法稳定性分析

对于长度为 n 的数组，冒泡排序需要经过 n(n-1)/2 次比较，最坏的情况下，即数组本身是倒序的情况下，需要经过 n(n-1)/2 次交换，所以其

> 冒泡排序的算法时间平均复杂度为 O(n²)。空间复杂度为 O(1)。

可以想象一下：如果两个相邻的元素相等是不会进行交换操作的，也就是两个相等元素的先后顺序是不会改变的。如果两个相等的元素没有相邻，那么即使通过前面的两两交换把两个元素相邻起来，最终也不会交换它俩的位置，所以相同元素经过排序后顺序并没有改变。

所以冒泡排序是一种稳定排序算法。所以冒泡排序是稳定排序。这也正是算法稳定性的定义：

> **排序算法的稳定性：通俗地讲就是能保证排序前两个相等的数据其在序列中的先后位置顺序与排序后它们两个先后位置顺序相同。**

**冒泡排序总结**：

1. 冒泡排序的算法时间平均复杂度为 O(n²)。
2. 空间复杂度为 O(1)。
3. 冒泡排序为稳定排序。

考虑到不少读者说每天代码量太多的问题，我们今天就只讲冒泡排序的 Java 实现，我们明天将带来三大简单排序的另外两种，**选择排序** 和 **插入排序**。



文章参考来源：https://juejin.im/post/5a96d6b15188255efc5f8bbd