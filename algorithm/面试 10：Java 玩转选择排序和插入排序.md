## 面试 10：Java 玩转选择排序和插入排序

昨天给大家讲解了 [Java 玩转冒泡排序](https://mp.weixin.qq.com/s/WFojXY4Ectfc4brNDp3SKg)，大家一定觉得并没有什么难度吧，不知道大佬们玩转了吗？不知道大家有没有多加思考，实际上在我们最后的一种思路上，还可以再继续改进。

我们先看看昨天最终版本的代码。

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

我们用一个 boolean 变量 `isSort` 来判断是否已经排序完成，当一整趟遍历都没有发生数据交换的时候，说明已经排序完成，直接 break 退出循环即可。

我们试想一下这样的场景：**假设有 100 个数字的数组，仅仅前 10 个无序，后面 90 个均有序并且都大于前面 10 个数字。**

我们采用上面的终极算法可以明显看到，第一趟排序后，最后发生交换的位置必定大于 10，且这个位置之后的数据必定已经有序了，但我们还是会去做徒劳的 90 次遍历，而且我们还要遍历 10 次！

显然我们可以找到这样的思路，**在第一次排序后，就记住最后发生交换的位置，第二次只要从数组头部遍历到这个位置就 OK 了。**

我们不妨直接看看代码实现：

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
        int flag = arr.length;
        int k;
        for (int i = 0; i < arr.length - 1; i++) {
            k = flag;
            flag = 0;
            for (int j = 1; j < k; j++) {
                if (arr[j - 1] > arr[j]) {
                    swap(arr, j - 1, j);
                    flag = j;
                }
            }
            if (flag == 0)
                break;
        }
    }

    public static void main(String[] args) {
        int[] arr = {6, 4, 1, 2, 3, 5, 7, 8, 9};
        bubbleSort(arr);
        printArr(arr);
    }
}
```

其实算法也就那么一回事儿，用心去理解它的原理，理解后，无论是用哪种语言实现起来都是非常简单的。那我们今天就来看看另外两种排序，选择排序和插入排序。

## 选择排序

**选择排序**（Selection sort）是一种简单直观的排序算法。选择排序之所以叫选择排序就是在一次遍历过程中找到最小元素的角标位置，然后把它放到数组的首端。我们排序过程都是在寻找剩余数组中的最小元素，所以就叫做选择排序。

它的思想如下：

1. 从待排序序列中，找到关键字最小的元素；起始假定第一个元素为最小
2. 如果最小元素不是待排序序列的第一个元素，将其和第一个元素互换；
3. 从余下的 N - 1 个元素中，找出关键字最小的元素，重复1，2步，直到排序结束。

![图片来源于网络](https://user-gold-cdn.xitu.io/2018/3/1/161e0ae4c72cb1b0?imageslim)

选择排序的主要优点与数据移动有关。如果某个元素位于正确的最终位置上，则它不会被移动。选择排序每次交换一对元素，它们当中至少有一个将被移到其最终位置上，因此对 n 个元素的表进行排序总共进行至多 n - 1 次交换。在所有的完全依靠交换去移动元素的排序方法中，选择排序属于非常好的一种。

我们来看看用 Java 是怎么实现的。

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

    private static void selectSort(int[] arr) {
        if (arr == null)
            return;
        int i, j, min, len = arr.length;
        for (i = 0; i < len - 1; i++) {
            min = i; // 未排序的序列中最小元素的下标
            for (j = i + 1; j < len; j++) {
                //在未排序元素中继续寻找最小元素，并保存其下标
                if (arr[min] > arr[j]) {
                    min = j;
                }
            }
            if (min != i)
                swap(arr, min, i);
        }
    }

    public static void main(String[] args) {
        int[] arr = {6, 4, 2, 1, 8, 3, 7, 9, 5};
        selectSort(arr);
        printArr(arr);
    }
}
```

上述 java 代码可以看出我们除了交换元素并未开辟额外的空间，所以额外的空间复杂度为 O(1)。

对于时间复杂度而言，选择排序序冒泡排序一样都需要遍历 n(n-1)/2 次,但是相对于冒泡排序来说每次遍历只需要交换一次元素，这对于计算机执行来说有一定的优化。但是选择排序也是名副其实的慢性子，即使是有序数组，也需要进行 n(n-1)/2 次比较，所以其时间复杂度为 O(n²)。

即便无论如何也要进行 n(n-1)/2 次比较，选择排序仍是不稳定的排序算法，我们举一个例子如：序列 5 8 5 2 9， 我们知道第一趟选择第 1 个元素 5 会与 2 进行交换，那么原序列中两个 5 的相对先后顺序也就被破坏了。

 **选择排序总结：**

1. 选择排序的算法时间平均复杂度为O(n²)。
2. 选择排序空间复杂度为 O(1)。
3. 选择排序为不稳定排序。

## 插入排序

对于插入排序，大部分资料都是使用扑克牌整理作为例子来引入的，我们打牌都是一张一张摸牌的，每摸到一张牌就会跟手里所有的牌比较来选择合适的位置插入这张牌，这也就是直接插入排序的中心思想，我们先来看下动图：

 ![图片来源于网络](https://user-gold-cdn.xitu.io/2018/3/1/161e0ae55a54d8ca?imageslim) 

 相信大家看完动图以后大概知道了插入排序的实现思路了。那么我们就来说下插入排序的思想。

#### 插入排序的思想

1. 从第一个元素开始，该元素可以认为已经被排序
2. 取出下一个元素，在已经排序的元素序列中从后向前扫描
3. 如果该元素（已排序）大于新元素，将该元素移到下一位置
4. 重复步骤 3，直到找到已排序的元素小于或者等于新元素的位置
5. 将新元素插入到该位置后
6. 重复步骤 2~5

 理解上述思想其实并不难，我们来看看用 Java 怎么实现：

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

    private static void insertionSort(int[] arr) {
        if (arr == null)
            return;
        int j;
        int temp;
        for (int i = 1; i < arr.length; i++) {
            // 设置哨兵，拿出待插入的值
            temp = arr[i];
            j = i;
            // 然后寻找正确插入的位置
            while (j > 0 && arr[j - 1] > temp) {
                arr[j] = arr[j - 1];
                j--;
            }
            arr[j] = temp;
        }
    }

    public static void main(String[] args) {
        int[] arr = {6, 4, 2, 1, 8, 3, 7, 9, 5};
        insertionSort(arr);
        printArr(arr);
    }
}
```

#### 插入排序的时间复杂度和空间复杂度分析

对于插入的时间复杂度和空间复杂度，通过代码就可以看出跟选择和冒泡来说没什么区别同属于 O(n²) 级别的时间复杂度算法 ，只是遍历方式有原来的 n n-1 n-2 ... 1，变成了 1 2 3 ... n 了。最终得到时间复杂度都是 n(n-1)/2。

对于稳定性来说，插入排序和冒泡一样，并不会改变原有的元素之间的顺序，如果遇见一个与插入元素相等的，那么把待插入的元素放在相等元素的后面。所以，相等元素的前后顺序没有改变，从原无序序列出去的顺序仍是排好序后的顺序，所以插入排序是稳定的。

对于插入排序这里说一个非常重要的一点就是：由于这个算法可以提前终止内层比较（ arr[j-1] > arr[j]）所以这个排序算法很有用！因此对于一些 NlogN 级别的算法，后边的归并和快速都属于这个级别的，算法来说对于 n 小于一定级别的时候（Array.sort 中使用的是47）都可以用插入算法来优化,另外对于近乎有序的数组来说这个提前终止的方式就显得更加又有优势了。

**插入排序总结：**

1. 插入排序的算法时间平均复杂度为O(n²)。
2. 插入排序空间复杂度为 O(1)。
3. 插入排序为稳定排序。
4. 插入排序对于近乎有序的数组来说效率更高，插入排序可用来优化高级排序算法

到现在，我们的三种简单排序就告一段落了，下面我们将直接进入 **归并排序** 和 **快速排序** 的讲解。这两个算法也是面试上的常客了，所以你准备好了么？

文章参考来源：https://juejin.im/post/5a96d6b15188255efc5f8bbd

 

 

 

 

 

 