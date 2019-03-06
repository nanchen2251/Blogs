## 面试：用 Java 实现一个 Singleton 模式

面试系列更新后，终于迎来了我们的第一期，我们也将贴近《剑指 Offer》的题目给大家带来 Java 的讲解，个人还是非常推荐《剑指 Offer》作为面试必刷的书籍的，这不，再一次把这本书分享给大家，PDF 版本在公众号后台回复「剑指Offer」即可获取。

我们在面试中总会遇到不少设计模式的问题，而设计模式中的 Singleton 模式又是我们最容易出现的考题，大多数人可能在此前已经有充分的了解，但不少人仅仅是停留在比较浅显的层次，今天我们就结合《剑指 Offer》给大家带来更加深入的讲解。

#### 题目：请用 Java 手写一个单例模式代码，希望尽可能考虑地全面。

不论是 Java 还是 Android 中单例模式肯定是我们经常用到的，所以这道题可能大多数人会第一时间想到饿汉式代码。

```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

上面是典型的饿汉式写法，因为单例的实例被声明成 static 和 final 变量了，所以在第一次加载类到内存中时就会初始化，所以也不会存在多线程问题，但它的缺点非常显而易见，也经常为人诟病。这明显不是一种懒加载模式（lazy initialization），就因为它是 static 和 final 的，所以类会在加载后就被初始化，导致我们代码的健壮性很差，假如后面更改需求，希望在 `getInstance()` 之前调用某个方法给它设置参数，这个就明显不符合使用场景了，面试官极有可能在看到这个代码后觉得你就是一个只知道完成功能没有大局观的人。

当然还会有不少人直接采用我们的懒汉式代码，这样就解决了延展性和懒加载了。

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

}
```

上述代码可能是大多数面试者的解法，包括教科书上也是这么教我们的，但这段代码却存在了一个致命的问题，那就是当多个线程并行调用 `getInstance()` 的时候，就会创建多个实例，这显然违背了面试官的意思。正好面试官加了一句希望尽可能考虑地全面，所以这样的代码肯定不能虏获面试官的芳心。

既然要线程安全，那我直接加锁呗。于是并有了下面的代码。他们也是懒汉式的，只不过线程安全了。

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {
    }

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

}
```

这样的解法实现了线程安全，但它并不是那么高效，因为在任何时候只能有一个线程去调用 `getInstance()` 方法，但实际上加锁操作也是耗时的，我们应该尽量地避免使用它。所以自然就引出了双重检验锁。

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

}
```

这段代码看起来很完美，很可惜，它是有问题。主要在于instance = new Singleton()这句，这并非是一个原子操作，事实上在 JVM 中这句话大概做了下面 3 件事情。

1. 给 instance 分配内存
2. 调用 Singleton 的构造函数来初始化成员变量
3. 将 instance 对象指向分配的内存空间（执行完这步 instance 就为非 null 了）

但是在 JVM 的即时编译器中存在指令重排序的优化。也就是说上面的第二步和第三步的顺序是不能保证的，最终的执行顺序可能是 1-2-3 也可能是 1-3-2。如果是后者，则在 3 执行完毕、2 未执行之前，被线程二抢占了，这时 instance 已经是非 null 了（但却没有初始化），所以线程二会直接返回 instance，然后使用，然后顺理成章地报错。

我们只需要将 instance 变量声明成 volatile 就可以了。

```java
public class Singleton {
    private volatile static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

}
```

有些人认为使用 `volatile` 的原因是可见性，也就是可以保证线程在本地不会存有 instance 的副本，每次都是去主内存中读取。但其实是不对的。使用 `volatile` 的主要原因是其另一个特性：禁止指令重排序优化。也就是说，在 `volatile` 变量的赋值操作后面会有一个内存屏障（生成的汇编代码上），读操作不会被重排序到内存屏障之前。比如上面的例子，取操作必须在执行完 1-2-3 之后或者 1-3-2 之后，不存在执行到 1-3 然后取到值的情况。从「先行发生原则」的角度理解的话，就是对于一个 volatile 变量的写操作都先行发生于后面对这个变量的读操作（这里的“后面”是时间上的先后顺序）。

但是特别注意在 Java 5 以前的版本使用了 `volatile` 的双检锁还是有问题的。其原因是 Java 5 以前的 JMM （Java 内存模型）是存在缺陷的，即时将变量声明成 volatile 也不能完全避免重排序，主要是 volatile 变量前后的代码仍然存在重排序问题。这个 `volatile` 屏蔽重排序的问题在 Java 5 中才得以修复，所以在这之后才可以放心使用 `volatile`。

那么，有没有一种既有懒加载，又保证了线程安全，还简单的方法呢？

当然有，静态内部类，就是一种我们想要的方法。我们完全可以把 Singleton 实例放在一个静态内部类中，这样就避免了静态实例在 Singleton 类加载的时候就创建对象，并且由于静态内部类只会被加载一次，所以这种写法也是线程安全的。

```java
public class Singleton {
    private static class Holder {
        private static Singleton INSTANCE = new Singleton();
    }

    private Singleton() {
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

这是我比较推荐的解法，这种写法用 JVM 本身的机制保证了线程安全的问题，同时读取实例的时候也不会进行同步，没什么性能缺陷，还不依赖 JDK 版本。

虽说如此，但看《Effective Java》中第三点来说，还是有必要提醒一下：**享有特权的客户端可以借助 `AccessibleObject.setAccessible` 方法，通过反射机制来调用私有构造器。**如果需要抵御这种攻击，可以修改构造器，让它在被要求创建第二个实例的时候抛出异常。

> 《Effective Java 中文版》PDF 在公众号后台回复「Effective Java」即可获取。

#### 我们其实还有更简单的枚举单例。

用过枚举写单例的人都说：**用枚举写单例真是太简单了。**下面的这段代码就是声明枚举单例的通常做法。

```java
public enum EasySingleton{
    INSTANCE;
}
```

这是从 Java 1.5 发行版本后就可以实用的单例方法，我们可以通过 `EasySingleton.INSTANCE` 来访问实例，这比调用 `getInstance()` 方法简单多了。创建枚举默认就是线程安全的，所以不需要担心 double checked locking，而且还能防止反序列化导致重新创建新的对象。但是还是很少看到有人这样写，可能是因为不太熟悉吧。

### 总结

一个总结肯定是必不可少的，上面也只是列举了我们常见的单例实现方式。当然也不完全，比如我们还可以用 static 代码块的方式实现懒汉式代码，但这里就不一一例举了。

就我个人而言，我还是比较推荐用静态内部类的方式使用单例模式，如果涉及到反序列化创建对象的话，不妨也试试枚举呗~