## 面试：用 Java 逆序打印链表

昨天的 Java 实现单例模式 中，我们的双重检验锁机制因为指令重排序问题而引入了 `volatile` 关键字，不少朋友问我，到底为啥要加 `volatile` 这个关键字呀，而它，到底又有什么神奇的作用呢？

对 `volatile` 这个关键字，在昨天的讲解中我们简单说了一下：被 `volatile` 修饰的共享变量，都会具有下面两个属性：

- 保证不同线程对该变量操作的内存可见性。
- 禁止指令重排序。

共享变量：如果一个变量在多个线程的工作内存中都存在副本，那么这个变量就是这几个线程的共享变量。

可见性：一个线程对共享变量值的修改，能够及时地被其它线程看到。

对于重排序，不熟悉的建议直接 Google 一下，这里也就不多提了。只需要记住，在多线程中操作一个共享变量的时候，一定要记住加上 `volatile` 修饰即可。

由于时间关系，我们还是得先进入今天的正题，对于 `volatile` 关键字，在要求并发编程能力的面试中还是很容易考察到的，后面我也会简单给大家讲解。

#### 输入一个单链表的头结点，从尾到头打印出每个结点的值。

这是《剑指 Offer》上的第五道面试题，链表是经常在面试中考察的一种数据结构，所以推荐大家一定要掌握。对于链表不熟悉的小伙伴可一定要去《大话数据结构》好好补课哟~

> 《剑指 Offer》 PDF 版本在公众号后台回复「剑指Offer」即可获取。
>
> 《大话数据结构》PDF 版本在公众号后台回复「大话数据结构」即可获取。

我们的链表有很多，单链表，双向链表，环链表等。这里是最普通的单链表模式，我们一般会在数据存储区域存放数据，然后有一个指针指向下一个结点。虽然 Java 中没有指针这个概念，但 Java 的引用恰如其分的填补了这个问题。

看到这道题，我们往往会很快反应到每个结点都有 next 属性，所以要从头到尾输出很简单。于是我们自然而然就会想到先用一个 `while` 循环取出所有的结点存放到数组中，然后再通过逆序遍历这个数组，即可实现逆序打印单链表的结点值。

我们假定结点的数据为 int 型的。实现代码如下：

```java
public class Test05 {
    public static class Node {
        int data;
        Node next;
    }

    public static void printLinkReverse(Node head) {
        ArrayList<Node> nodes = new ArrayList<>();
        while (head != null) {
            nodes.add(head);
            head = head.next;
        }
        for (int i = nodes.size() - 1; i >= 0; i--) {
            System.out.print(nodes.get(i).data + " ");
        }
    }

    public static void main(String[] args) {
        Node head = new Node();
        head.data = 1;
        head.next = new Node();
        head.next.data = 2;
        head.next.next = new Node();
        head.next.next.data = 3;
        head.next.next.next = new Node();
        head.next.next.next.data = 4;
        head.next.next.next.next = new Node();
        head.next.next.next.next.data = 5;
        printLinkReverse(head);
    }
}
```

这样的方式确实能实现逆序打印链表的数据，但明显用了整整两次循环，时间复杂度为 O(n²)。等等！逆序输出？似乎有这样一个数据结构可以完美解决这个问题，这个数据结构就是栈。

栈是一种「后进先出」的数据结构，用栈的原理更好能达到我们的要求，于是实现代码如下：

```java
public class Test05 {
    public static class Node {
        int data;
        Node next;
    }

    public static void printLinkReverse(Node head) {
        Stack<Node> stack = new Stack<>();
        while (head != null) {
            stack.push(head);
            head = head.next;
        }
        while (!stack.isEmpty()) {
            System.out.print(stack.pop().data + " ");
        }
    }

    public static void main(String[] args) {
        Node head = new Node();
        head.data = 1;
        head.next = new Node();
        head.next.data = 2;
        head.next.next = new Node();
        head.next.next.data = 3;
        head.next.next.next = new Node();
        head.next.next.next.data = 4;
        head.next.next.next.next = new Node();
        head.next.next.next.next.data = 5;
        printLinkReverse(head);
    }
}
```

既然可以用栈来实现，我们也极容易想到递归也能解决这个问题，因为递归本质上也就是一个栈结构。要实现逆序输出链表，我们每访问一个结点的时候，我们先递归输出它后面的结点，再输出该结点本身，这样链表的输出结果自然也是反过来了。

代码如下：

```java
public class Test05 {
    public static class Node {
        int data;
        Node next;
    }

    public static void printLinkReverse(Node head) {
        if (head != null) {
            printLinkReverse(head.next);
            System.out.print(head.data+" ");
        }
    }

    public static void main(String[] args) {
        Node head = new Node();
        head.data = 1;
        head.next = new Node();
        head.next.data = 2;
        head.next.next = new Node();
        head.next.next.data = 3;
        head.next.next.next = new Node();
        head.next.next.next.data = 4;
        head.next.next.next.next = new Node();
        head.next.next.next.next.data = 5;
        printLinkReverse(head);
    }
}
```

虽然递归代码看起来确实很整洁，但有个问题：当链表非常长的时候，一定会导致函数调用的层级很深，从而有可能导致函数调用栈溢出。所以显示用栈基于循环实现的代码，健壮性还是要好一些的。

好了，今天的面试讲解就到这，我们明天再见！

