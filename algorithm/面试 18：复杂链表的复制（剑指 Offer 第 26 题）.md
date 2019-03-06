## 面试 18：复杂链表的复制（剑指 Offer 第 26 题）

在上一篇推文中，我们留下的习题是来自《剑指 Offer》 的面试题 26：复杂链表的复制。

> 请实现复杂链表的复制，在复杂链表中，每个结点除了 next 指针指向下一个结点外，还有一个 sibling 指向链表中的任意结点或者 NULL。比如下图就是一个含有 5  个结点的复杂链表。

![image-20180728122908757](/var/folders/6m/5yg4nys56t1dd5xpwk68cmbw0000gn/T/abnerworks.Typora/image-20180728122908757.png)

## 提前想好测试用例

依旧是我们熟悉的第一步，先想好我们的测试用例：

1. 输入一个 null ，期望什么也不输出；
2. 输入一个结点，sibling 指向自身，期望打印符合题干的值；
3. 输入多个结点，部分 sibling 指向 null，期望打印符合题干的值。

## 思考程序逻辑

测试用例思考完毕，自然是开始思考我们的测试逻辑了，在思考的过程中，我们不妨尝试和面试官进行沟通，这样可以避免我们走不少弯路，而且也容易给面试官留下一个善于思考和沟通的好印象。

极易想到的逻辑是，**我们先复制我们传统的单链表，然后再遍历单链表，复制 sibling 的指向。**

假设链表中有个结点 A，A 的 sibling 指向结点 B，这个 B 可能在 A 前面也可能在 A 后面，所以我们唯一的办法只有从头结点开始遍历。对于一个含有 n 个结点的链表，由于定位每个结点的 sibling 都需要从链表头结点开始经过 O(n) 步才能找到，因此这种方法的时间复杂度是 O(n²)。

当我们告知面试官我们这样的思路的时候，面试官告诉我们，他期待的并不是这样的算法，这样的算法时间复杂度也太高了，希望能有更加简单的方式。

得到了面试官的诉求，我们再来看看我们前面的想法时间都花在哪儿去了。

很明显，我们上面的想法在定位 sibling 指向上面花了大量的时间，我们可以尝试在这上面进行优化。我们还是分为两步：第一步仍然是先复制原始链表上的每个结点 N 创建 N1，然后把这些创建出来的结点用 next 连接起来。同时我们把 <N,N1> 的配对信息放在一个哈希表中。第二步是设置复制链表的 sibling 指向，如果原始链表中有 N 指向 S，那么我们的复制链表中必然存在 N1 指向 S1 。由于有了哈希表，我们可以用 O(1) 的时间，根据 S 找到 S1。

这样的方法降低了时间成本，我们高兴地与面试官分享我们的想法，却被面试官指出，这样的想法虽然把时间复杂度降低到了 O(n)，但却由于哈希表的存在，需要 O(n) 的空间，而他所期望的方法是不占用任何辅助空间的。

接下来我们再换一下思路，不用辅助空间，我们却要用更少的实际解决 sibling 的指向问题。

我们前面似乎对于指向都采用过两个指针的方法，这里似乎可以用类似的处理方式处理。

我们不妨利用原有链表对每个结点 N 在后面直接在后面创建 N1，这样相当于我们扩长原始链表长度为现有链表的 2 倍，奇数位置的结点连接起来是原始链表，偶数位置的结点连接起来就是我们的复制链表。

## 开始编写代码

我们先完成第一部分的代码。根据原始链表的每个结点 N ，创建 N1，并把 N 的 next 指向 N1，N1 的 next 指向 N 的 next。

```java
private static void cloneNodes(Node head) {
    Node node = null;
    while (head != null) {
        // 先新建结点
        node = new Node(head.data);
        // 再把head 的 next 指向 node 的 next
        node.next = head.next;
        // 然后把 node 作为 head 的 next
        head.next = node;
        // 最后遍历条件
        head = node.next;
    }
}
```

上面完成了复制结点，下面我们需要编写 sibling 的指向复制。

我们的思想是：当 N 执行 S，那么 N1 就应该指向 S1，即 N.next.sibling = N.sibling.next；

```java
private static void connectNodes(Node head) {
    while (head != null) {
        if (head.sibling != null) {
            //如果 当前结点的 sibling 不为 null,那就把它后面的复制结点指向当前sibling指向的下一个结点
            head.next.sibling = head.sibling.next;
        }
        // 遍历
        head = head.next.next;
    }
}
```

最后我们只需要拿出原本的链表（奇数）和复制的链表（偶数）即可。

```java
private static Node reconnectList(Node head) {
    if (head == null)
        return null;
    // 用于存放复制链表的头结点
    Node cloneHead = head.next;
    // 用于记录当前处理的结点
    Node temp = cloneHead;
    // head 的 next 还是要指向原本的 head.next
    // 实际上现在由于复制后，应该是 head.next.next，即cloneHead.next
    head.next = cloneHead.next;
    // 指向新的被复制结点
    head = head.next;
    while (head != null) {
        // temp 代表的是复制结点
        // 先进行赋值
        temp.next = head.next;
        // 赋值结束应该给 next 指向的结点赋值
        temp = temp.next;
        // head 的下一个结点应该指向被赋值的下一个结点
        head.next = temp.next;
        head = temp.next;
    }
    return cloneHead;
}
```

合并后的最终代码就是：

```java
public class Test18 {

    private static class Node {
        int data;
        Node next;
        Node sibling;

        Node(int data) {
            this.data = data;
        }
    }

    private static Node complexListNode(Node head) {
        if (head == null)
            return null;
        // 第一步，复制结点，并用 next 连接
        cloneNodes(head);
        // 第二步，把 sibling 也复制起来
        connectNodes(head);
        // 第三步，返回偶数结点，连接起来就是复制的链表
        return reconnectList(head);
    }

    private static void cloneNodes(Node head) {
        Node node = null;
        while (head != null) {
            // 先新建结点
            node = new Node(head.data);
            // 再把head 的 next 指向 node 的 next
            node.next = head.next;
            // 然后把 node 作为 head 的 next
            head.next = node;
            // 最后遍历条件
            head = node.next;
        }
    }

    private static void connectNodes(Node head) {
        while (head != null) {
            if (head.sibling != null) {
                // 如果 当前结点的 sibling 不为 null,那就把它后面的复制结点指向当前sibling指向的下一个结点
                head.next.sibling = head.sibling.next;
            }
            // 遍历
            head = head.next.next;
        }
    }

    private static Node reconnectList(Node head) {
        if (head == null)
            return null;
        // 用于存放复制链表的头结点
        Node cloneHead = head.next;
        // 用于记录当前处理的结点
        Node cloneNode = cloneHead;
        // head 的 next 还是要指向原本的 head.next
        // 实际上现在由于复制后，应该是 head.next.next，即cloneHead.next
        head.next = cloneHead.next;
        // 因为我们第一个结点已经拆分了，所以需要指向新的被复制结点才可以开始循环
        head = head.next;
        while (head != null) {
            // cloneNode 代表的是复制结点
            // 先进行赋值
            cloneNode.next = head.next;
            // 赋值结束应该给 next 指向的结点赋值
            cloneNode = cloneNode.next;
            // head 的下一个结点应该指向被赋值的下一个结点
            head.next = cloneNode.next;
            head = cloneNode.next;
        }
        return cloneHead;
    }


    public static void main(String[] args) {
        Node head1 = new Node(1);
        Node node2 = new Node(2);
        Node node3 = new Node(3);
        Node node4 = new Node(4);
        Node node5 = new Node(5);
        head1.next = node2;
        node2.next = node3;
        node3.next = node4;
        node4.next = node5;
        node5.next = null;
        head1.sibling = node4;
        node2.sibling = null;
        node3.sibling = node5;
        node4.sibling = node2;
        node5.sibling = head1;

        print(head1);
        Node root = complexListNode(head1);
        System.out.println();
        print(head1);
        print(root);
        System.out.println();
        System.out.println(isSameLink(head1, root));
    }

    private static boolean isSameLink(Node head, Node root) {
        while (head != null && root != null) {
            if (head == root) {
                head = head.next;
                root = root.next;
            } else {
                return false;
            }
        }
        return head == null && root == null;
    }

    private static void print(Node head) {
        Node temp = head;
        while (head != null) {
            System.out.print(head.data + "->");
            head = head.next;
        }
        System.out.println("null");
        while (temp != null) {
            System.out.println(temp.data + "=>" + (temp.sibling == null ? "null" : temp.sibling.data));
            temp = temp.next;
        }
    }
}
```

## 验证测试用例

写毕代码，我们验证我们的测试用例。

1. 输入一个 null ，也不会输出，测试通过；
2. 输入一个结点，sibling 指向自身，测试通过；
3. 输入多个结点，部分 sibling 指向 null，测试通过。

## 课后习题

下一次推文的习题来自于《剑指 Offer》第 29 题：数组中超过一半的数字

> 面试题：数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字并输出。比如 {1,2,3,2,2,2,1} 中 2 的次数是 4，数组长度为 7，所以输出 2。要求不能修改输入的数组。

