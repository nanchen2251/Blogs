## 面试 5：手写 Java 的 pow() 实现。

我们在处理一道编程面试题的时候，通常除了注意代码规范以外，千万要记得自己心中模拟一个单元测试。主要通过三方面来处理。

- 功能性测试
- 边界值测试
- 负面性测试

不管如何，一定要保证自己代码考虑的全面，而不要简单地猜想用户的输入一定是正确的，只是去实现功能。通常你编写一个能接受住考验的代码，会让面试官对你刮目相看，你可以不厉害，但已经充分说明了你的靠谱。

今天我们的面试题目是：

> **面试题：尝试实现 Java 的 Math.pow(double base,int exponent) 函数算法，计算 base 的 exponent 次方，不得使用库函数，同时不需要考虑大数问题。**
>
> > 面试题来源于《剑指 Offer》第 11 题，数字的整数次方。
> >
> > 不要介意 Java 真正的方法是 Math.pow(double var1,double var2)。

由于不需要考虑大数问题，不少小伙伴心中暗自窃喜，这题目也太简单了，给我撞上了，运气真好，于是直接写出下面的代码：

```java
public class Test11 {

    private static double power(double base, int exponent) {
        double result = 1.0;
        for (int i = 0; i < exponent; i++) {
            result *= base;
        }
        return result;
    }

    public static void main(String[] args) {
        System.out.println(power(2, 2));
        System.out.println(power(2, 4));
        System.out.println(power(3, 1));
        System.out.println(power(3, 0));
    }
}
```

写的快自然是好事，如果正确的话会被面试官认为是思维敏捷。但如果考虑不周的话，恐怕就极容易被面试官认为是不靠谱的人了。在技术能力和靠谱度之间，大多数面试官更青睐于靠谱度。

我们上面确实做到了功能测试，但面试官可能会直接提示我们，假设我们的 `exponent` 输入一个负值，能得到正确值么？

跟着自己的代码走一遍，终于意识到了这个问题，当 `exponent` 为负数的时候，循环根本就进不去，无论输入的负数是什么，都会返回 1.0，这显然是不正确的算法。

我们在数学中学过，**给一个数值上负数次方，相当于给这个数值上整数次方再求倒数。**

意识到这点，我们修正一下代码。

```java
public class Test11 {

    private static double power(double base, int exponent) {
        // 因为除了 0 以外，任何数值的 0 次方都为 1，所以我们默认为 1.0；
        // 0 的 0 次方，在数学书是没有意义的，为了贴切，我们也默认为 1.0
        double result = 1.0;
        // 处理负数次方情况
        boolean isNegetive = false;
        if (exponent < 0) {
            isNegetive = true;
            exponent = -exponent;
        }
        for (int i = 0; i < exponent; i++) {
            result *= base;
        }
        if (isNegetive)
            return 1 / result;
        return result;
    }

    public static void main(String[] args) {
        System.out.println(power(2, 2));
        System.out.println(power(2, 4));
        System.out.println(power(3, 1));
        System.out.println(power(3, -1));
    }
}
```

我们在代码中增加了一个判断是否为负数的 `isNegetive` 变量，当为负数的时候，我们就置为 true，并计算它的绝对值次幂，最后返回结果的时候返回它的倒数。

面试官看到这样的代码，可能就有点按捺不住内心的怒火了，不过由于你此前一直面试回答的较好，也打算再给你点机会，面试官提示你，当 `base` 传入 0，`exponent` 传入负数，会怎样？

瞬间发现了自己的问题，这不是犯了数学最常见的问题，给 0 求倒数么？

> 虽然 Java 的 Math.pow() 方法也存在这个问题，但我们这里忽略不计。

于是马上更新代码。

```java
public class Test11 {


    private static double power(double base, int exponent) {
        // 因为除了 0 以外，任何数值的 0 次方都为 1，所以我们默认为 1.0；
        // 0 的 0 次方，在数学书是没有意义的，为了贴切，我们也默认为 1.0
        double result = 1.0;
        // 处理底数为 0 的情况，底数为 0 其他任意次方结果都应该是 0
        if (base == 0)
            return 0.0;
        // 处理负数次方情况
        boolean isNegetive = false;
        if (exponent < 0) {
            isNegetive = true;
            exponent = -exponent;
        }
        for (int i = 0; i < exponent; i++) {
            result *= base;
        }
        if (isNegetive)
            return 1 / result;
        return result;
    }

    public static void main(String[] args) {
        System.out.println(power(2, 2));
        System.out.println(power(2, 4));
        System.out.println(power(3, 1));
        System.out.println(power(0, -1));
    }
}
```

有了上一次的经验，这次并不敢直接上交代码了，而是认真检查边界值和各种情况。检查 1 遍，2 遍，均没有发现问题，提交代码。

> 计算机表示小数均有误差，这个在 Python 中尤其严重，但经数次测试，《剑指 Offer》中讲的双精度误差问题似乎在 Java 的 == 运算符中并不存在。如有问题，欢迎指正。

上面的代码基本还算整，健壮性也还不错，但面试官可能还想问问有没有更加优秀的算法。

仔细查看，确实似乎是有办法优化的，比如我们要求 `power(2,16)` 的值，我们只需要先求出 2 的 8 次方，再平方就可以了；以此类推，我们计算 2 的 8 次方的时候，可以先计算 2 的 4 次方，然后再做平方运算.....妙哉妙哉！

需要注意的是，如果我们的幂数为奇数的话，我们需要在最后再乘一次我们的底数。

我们尝试修改代码如下：

```java
public class Test11 {
    private static double power(double base, int exponent) {
        // 因为除了 0 以外，任何数值的 0 次方都为 1，所以我们默认为 1.0；
        // 0 的 0 次方，在数学书是没有意义的，为了贴切，我们也默认为 1.0
        double result = 1.0;
        // 处理底数为 0 的情况，底数为 0 其他任意次方结果都应该是 0
        if (base == 0)
            return 0.0;
        // 处理负数次方情况
        boolean isNegetive = false;
        if (exponent < 0) {
            isNegetive = true;
            exponent = -exponent;
        }
        result = getTheResult(base, exponent);
        if (isNegetive)
            return 1 / result;
        return result;
    }

    private static double getTheResult(double base, int exponent) {
        // 如果指数为0，返回1
        if (exponent == 0) {
            return 1;
        }
        // 指数为1，返回底数
        if (exponent == 1) {
            return base;
        }
        // 递归求一半的值
        double result = getTheResult(base, exponent >> 1);
        // 求最终值，如果是奇数，还要乘一次底数
        result *= result;
        if ((exponent & 0x1) == 1) {
            result *= base;
        }
        return result;

    }

    public static void main(String[] args) {
        System.out.println(power(2, 2));
        System.out.println(power(2, 4));
        System.out.println(power(3, -1));
        System.out.println(power(0.1, 2));
    }
}
```

完美解决。

在提交代码的时候，还可以主动提示面试官，我们在上面用右移运算符代替了除以 2，用位与运算符代替了求余运算符 % 来判断是一个奇数还是一个偶数。让他知道我们对编程的细节真的很重视，这大概也就是细节决定成败吧。一两个细节的打动说不定就让面试官下定决心给我们发放 Offer 了。

> **位运算的效率比乘除法及求余运算的效率要高的多**。
>
> 因为移位指令占 2 个机器周期，而乘除法指令占 4 个机器周期。从硬件上看，移位对硬件更容易实现，所以我们更优先用移位。

好了，今天我们的面试精讲就到这里，我们明天再见！



