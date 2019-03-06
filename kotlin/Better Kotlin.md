## Better Kotlin

转眼间使用 Kotlin 已经有两个月了，时间不长，我也算搭上了 Google 宣布 Kotlin 作为官方支持语言的一波末班车。可能大家早已从纯 Java 开发  Android 转为了混合使用开发甚至是  Kotlin 开发，那你转向 Kotlin 的初衷又是什么呢？

对于我，很简单，只是因为一句话：「Google 爸爸都推荐的语言，我们没理由不用！」

Kotlin 有着诸多的特性，比如空指针安全、方法扩展、支持函数式编程、丰富的语法糖等。这些特性使得 Kotlin 的代码比 Java 简洁优雅许多，提高了代码的可读性和可维护性，节省了开发时间，提高了开发效率，但同样作为 Kotlin 使用者的你，我相信你一定也有不少小建议和小技巧，一直想迫不及待地分享给大家。

**那就给你一个机会，愿你把你的黑科技悄悄留言在本文下方！**截止到明天早上 9 点，点赞最多的找我有小奖励哟~

### 我想给大家的一些小建议

这么有趣的活动，那我作为一名两个月的 Kotlin 开发，自然也应该来这个活动凑凑热闹。

#### 1. 避免使用 IDE 自带的插件转换 Java 代码

想必 IDE 里面的插件 "Covert Java File To Kotlin File" 早已被大家熟知，要是不知道的小伙伴，赶紧写个 Java 文件，尝试点击 Android Studio 工具栏的 Code 下面的 "Convert Java File To Kotlin File"，看看都有什么小妙用。

这也是南尘最开始喜欢使用的方式，没有技术却有一颗装 ✘ 的内心，直接写成 Java 文件，再直接一键转换为 Kotlin。甚至宝宝想告诉你，我 GitHub 上 1k Star 的 [AiYaGilr](https://github.com/nanchen2251/AiYaGirl) 项目的 Kotlin 分支，也是这样而来。但真是踩了不少的坑。

这样的方式足够地快，但却会出现很多很多的 ```!!```，这是由于 Kotlin 的 null safety 特性。这是 Kotlin 在 Android 开发中的很牛逼的一大特性，想必不少小伙伴都被此 Android 的 ```NullPointException``` 困扰许久。我们直接转换 Java 文件造成的各种 ```!!``` ，其实也就意味着你可能存在潜在的未处理的 ```KotlinNullPointException```。

#### 2. 尽量地使用 val

`val` 是线程安全的，并且不需要担心 null 的问题，我们自然应该尽可能地使用它。

比如我们常用的 Android 解析的服务器数据，我们应该为自己的 data class 设置为 `val`，因为它本身就不应该是可写的。

当我第一次使用 Kotlin 的时候，我以为` val` 和 `var` 的区别在于` val` 代表不可变，而 `var` 代表是可变的。但事实比这更加微妙：**val 不代表不可变，val 意味着只读。**。这意味着你不允许明确声明为 `val`，它就不能保证它是不可变的。

对于普通变量来说，「不可变」和「只读」之间并没什么区别，因为你没办法复写一个 `val` 变量，所以在此时却是是不可变的。但在 class 的成员变量中，「只读」和「不可变」的区别就大了。

在 Kotlin 的类中，val 和 var 是用于表示属性是否有 getter/setter：

- var：同时有 getter 和 setter。
- val：只有 getter。

这里是可以通过自定义 getter 函数来返回不同的值：

```kotlin
class Person(val birthDay: DateTime) {  
  val age: Int
    get() = yearsBetween(birthDay, DateTime.now())
}
```

可以看到，虽然没有方法来设置 age 的值，但会随着当前日期的变化而变化。

这种情况下，我建议不要自定义 val 属性的 getter 方法。如果一个只读的类属性会随着某些条件而变化，那么应当用函数来替代：

```kotlin
class Person(val birthDay: DateTime) {  
  fun age(): Int = yearsBetween(birthDay, DateTime.now())
}
```

这也是 [Kotlin 代码约定 ](https://link.zhihu.com/?target=https%3A//kotlinlang.org/docs/reference/coding-conventions.html%23functions-vs-properties)中所提到的，当具有下面列举的特点时使用属性，不然更推荐使用函数：

- 不会抛出异常。
- 具有 O(1) 的复杂度。
- 计算时的消耗很少。
- 同时多次调用有相同的返回值。

因此上面提到的，自定义 getter 方法并随着当前时间的不同而返回不同的值违反了最后一条原则。大家也要尽量的避免这种情况。

#### 3. 你真的应该好好注意一下伴生对象

伴生对象通过在类中使用 `companion object` 来创建，用来替代静态成员，类似于 Java 中的静态内部类。所以在伴生对象中声明常量是很常见的做法，但如果写法不对，可能就会产生额外开销。

比如下面的这段代码：

```kotlin
class CompanionKotlin {
    companion object {
        val DATA = "CompanionKotlin_DATA"
    }

    fun getData(): String = DATA
}
```

挺简洁地一段代码。但将这段简洁的 Kotlin 代码转换为等同的 Java 代码后，却显的晦涩难懂。

```java
public final class CompanionKotlin {
   @NotNull
   private static final String DATA = "CompanionKotlin_DATA";
   public static final CompanionKotlin.Companion Companion = new CompanionKotlin.Companion((DefaultConstructorMarker)null);

   @NotNull
   public final String getData() {
      return DATA;
   }
	// ...
   public static final class Companion {
      @NotNull
      public final String getDATA() {
         return CompanionKotlin.DATA;
      }

      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```

与 Java 直接读取一个常量不同，Kotlin 访问一个伴生对象的私有常量字段需要经过以下方法：

- 调用伴生对象的静态方法
- 调用伴生对象的实例方法
- 调用主类的静态方法
- 读取主类中的静态字段

为了访问一个常量，而多花费调用4个方法的开销，这样的 Kotlin 代码无疑是低效的。

我们可以通过以下解决方法来减少生成的字节码：

1. 对于基本类型和字符串，可以使用 `const` 关键字将常量声明为编译时常量。
2. 对于公共字段，可以使用 `@JvmField` 注解。
3. 对于其他类型的常量，最好在它们自己的主类对象而不是伴生对象中来存储公共的全局常量。

#### 4. @JvmStatic、@JvmFiled 和 object 还有这种故事？

我们在 Kotlin 中发现了 `object` 这个东西，我以前就一直对这个东西很好奇，不知道这是个什么玩意儿。

> object ？难道又一个对象？

之前有人写过这样的代码，表示很不解，一个接口类型的成员变量，访问外部类的成员变量 name。这不是理所应当的么？

```kotlin
interface Runnable {
    fun run()
}

class Test {
    private val name: String = "nanchen"

    object impl : Runnable {
        override fun run() {
            // 这里编译器会报红报错。对 name
            println(name)
        }
    }
}
```

即使查看 Kotlin 官方文档，也有这样一段描述：

> Sometimes we need to create an object of a slight modification of some class, without explicitly declaring a new subclass for it. Java handles this case with anonymous inner classes. Kotlin slightly generalizes this concept with object expressions and object declarations.

核心意思是：Kotlin 使用 object 代替 Java 匿名内部类实现。

很明显，即便如此，这里的访问应该也是合情合理的。从匿名内部类中访问成员变量在 Java 语言中是完全允许的。

这个问题很有意思，解答这个我们需要生成 Java 字节码，再反编译成 Java 看看具体生成的代码是什么。

```java
public final class Test {
   private final String name = "nanchen";
   public static final class impl implements Runnable {
      public static final Test.impl INSTANCE;

      public void run() {
      }

      static {
         Test.impl var0 = new Test.impl();
         INSTANCE = var0;
      }
   }
}

public interface Runnable {
   void run();
}
```

静态内部类！确实，Java 中静态内部类是不允许访问外部类的成员变量的。但，说好的 object 代替的是 Java 的匿名内部类呢？那这里为啥是静态内部类。

这里一定要注意，如果你只是这样声明了一个object，Kotlin认为你是需要一个静态内部类。而如果你用一个变量去接收object表达式，Kotlin认为你需要一个匿名内部类对象。

因此，这个类应该这样改进：

```kotlin
interface Runnable {
    fun run()
}

class Test {
    private val name: String = "nanchen"

    private val impl = object : Runnable {
        override fun run() {
            println(name)
        }
    }
}
```

为了避免出现这个问题，谨记一个原则：如果 object 只是声明，它代表一个静态内部类。如果用变量接收 object 表达式，它代表一个匿名内部类对象。

讲到这，自然也就知道了 Kotlin 对 object 的三个作用：

- 简化生成静态内部类
- 生成匿名内部类对象
- 生成单例对象

咳咳，说了那么多，到底和 @JvmStatic 和 @JvmField 有啥关系呢?

实际上，目前我们大多数的 Android 项目都是 Java 和 Kotlin 混编的，包括我们的项目在内也是如此。所以我们总是免不了 Java 和 Kotlin 互调的情况。我们可能经常会在代码中这样编写：

```kotlin
object Test1 {
    val NAME = "nanchen"
    fun getAge() = 18
}
```

在 Java 中会调用是这样的：

```java
System.out.println("name:"+Test1.INSTANCE.getNAME()+",age:"+Test1.INSTANCE.getAge());
```

作为强迫症重度患者的我，自然是无法接受上面这样奇怪的代码。所以我强烈建议大家在 object 和 companion object 中分别为变量和方法增加上 @JvmField 和 @JvmStatic 注解。

```kotlin
object Test1 {
    @JvmField
    val NAME = "nanchen"
    @JvmStatic
    fun getAge() = 18
}
```

 这样外面 Java 调用起来就好看多了。

### 5. by lazy 和 lateinit 相爱相杀

在 Android 开发中，我们经常会有不少的成员变量需要在 onCreate() 中对其进行初始化，特别是我们在 XML 中使用的各种控件，而 Kotlin 要求声明成员变量的时候默认需要为它声明一个初始值。这时候就会出现不少的下面这样的代码。

```kotlin
private var textView:TextView? = null
```

迫于压力，我们不能不为这些 View 加上 ? 代表它们可以为空，然后为它们赋值为 null。实际上，我们在使用中一点都不希望它们为空。这样造成的后果就是，我们每次要使用它的时候都必须去先判断它不为空。这样无用的代码，无疑是浪费了我们的工作时间。

好在 Kotlin 推出了 lateinit 关键字：延迟加载。这样我们可以先绕过 kotlin 的强制要求，在后面使用的时候，再也不需要先判断它是否为空了。但要注意，访问未初始化的 *lateinit* 属性会导致*UninitializedPropertyAccessException*。

并且 *lateinit* 不支持基础数据类型，比如 Int。对于基础数据类型，我们可以这样：

```kotlin
private var mNumber: Int by Delegates.notNull<Int>()
```

当然，我们还可以使用 let 函数来进行上面的这种情况，但无疑都是画蛇添足的。

我们前面说了，在一些明知是只读不可写不可变的变量，我们尽可能地用 val 去修饰它。而 lateinit 仅仅能修饰 var 变量，所以 by lazy 懒加载，是时候表演真正的技术了。

对于很多不可变的变量，比如上个页面通过 bundle 传递过来的用于该页面请求网络的参数，比如 MVP 架构开发中的 Presenter，我们都应该用 by lazy 关键字去初始化它。

`lazy()` 委托属性可以用于只读属性的惰性加载，但是在使用 `lazy()` 时经常被忽视的地方就是有一个可选的model参数：

- LazyThreadSafetyMode.SYNCHRONIZED：初始化属性时会有双重锁检查，保证该值只在一个线程中计算，并且所有线程会得到相同的值。
- LazyThreadSafetyMode.PUBLICATION：多个线程会同时执行，初始化属性的函数会被多次调用，但是只有第一个返回的值被当做委托属性的值。
- LazyThreadSafetyMode.NONE：没有双重锁检查，不应该用在多线程下。

`lazy()` 默认情况下会指定 `LazyThreadSafetyMode.SYNCHRONIZED`，这可能会造成不必要线程安全的开销，应该根据实际情况，指定合适的model来避免不需要的同步锁。

### 6.注意 Kotlin 中的 for 循环

Kotlin提供了 `downTo`、`step`、`until`、`reversed` 等函数来帮助开发者更简单的使用 For 循环，如果单一的使用这些函数确实是方便简洁又高效，但要是将其中两个结合呢？比如下面这样：

```kotlin
class A {
    fun loop() {
        for (i in 10 downTo 0 step 3) {
            println(i)
        }
    }
}
```

上面使用了 downTo 和 step 两个关键字，我们看看 Java 是怎样实现的。

```java
public final class A {
   public final void loop() {
      IntProgression var10000 = RangesKt.step(RangesKt.downTo(10, 0), 3);
      int i = var10000.getFirst();
      int var2 = var10000.getLast();
      int var3 = var10000.getStep();
      if (var3 > 0) {
         if (i > var2) {
            return;
         }
      } else if (i < var2) {
         return;
      }

      while(true) {
         System.out.println(i);
         if (i == var2) {
            return;
         }

         i += var3;
      }
   }
}
```

毫无疑问：`IntProgression var10000 = RangesKt.step(RangesKt.downTo(10, 0), 3);` 一行代码就创建了两个 `IntProgression` 临时对象，增加了额外的开销。

### 7. 注意 Kotlin 的可空和不可空

最近闹了一个笑话，在项目中需要写一个上传跳绳数据的功能。于是有了下面的代码。

```java
public interface ISkipService {
    /**
     * 上传用户跳绳数据
     */
    @POST("v2/rope/upload_jump_data")
    Observable<BaseResponse<Object>> uploadJumpData(@Field("data") List<SkipHistoryBean> data);
}
```

写毕上面的接口，我们再到 ViewModel 中进行网络请求。

```java
private List<SkipHistoryBean> list = new ArrayList<>();

public void uploadClick() {
    mNavigator.showProgressDialog();
    list.add(bean);
    RetrofitManager.create(ISkipService.class)
        .uploadJumpData(list)
        .compose(RetrofitUtil.schedulersAndGetData())
        .subscribe(new BaseSubscriber<Object>() {
            @Override
            protected void onSuccess(Object data) {
                mNavigator.hideProgressDialog();
                mNavigator.uploadDataSuccess();
                // 点击上传成功，删除数据库
                deleteDataFromDB();
            }

            @Override
            protected void onFail(ErrorBean errorBean) {
                super.onFail(errorBean);
                mNavigator.hideProgressDialog();
                mNavigator.uploadDataFailed(errorBean.error_description);
            }
        });
}
```

运行其实并没有什么问题。但由于某些原因，当我把上面的 ISkipService 类修改为了 Kotlin 实现，却发生了崩溃，从代码上暂时没看出问题。

```kotlin
interface ISkipService {
    /**
     * 上传用户跳绳数据
     */
    @POST("v2/rope/upload_jump_data")
    fun uploadJumpData(@Field("data") data: List<SkipHistoryBean>): Observable<BaseResponse<Any>>
}
```

但确实就是崩溃了。仔细一看，发现 Java 编写这个接口的时候，会被认为这个参数 "data" 对应的 "value" 是可以为 null 的，而改为 Kotlin 后，由于 Kotlin 默认不为空的机制，所以需要的参数是一个不可以为 null 的 List 集合。而我们的 ViewModel 中使用的 Java 代码，由于 Java 认为我们的 List 是可以为 null 的，所以导致了类型不匹配的崩溃。

找到了原因，解决方案也就很简单，在 Kotlin 接口中允许参数 data 为 null 或者直接在调用点加上 @NotNull 注解即可。

### 写在最后

真是想继续写呀，但参考了不少的资料，大家如果觉得有意思可以尽情地参见原文。

> 参考链接：
> https://blog.danlew.net/2017/05/30/mutable-vals-in-kotlin/
> https://juejin.im/post/5ad18d705188255c5668ddf0
> https://tech.meituan.com/Kotlin_code_inspect.html

 

 

 