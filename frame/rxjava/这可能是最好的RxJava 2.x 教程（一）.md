 

这可能是最好的 RxJava 2.x 入门教程系列专栏
文章链接：
[这可能是最好的 RxJava 2.x 入门教程(完结版)](http://www.jianshu.com/p/0cd258eecf60)**【重磅推出】**
[这可能是最好的RxJava 2.x 入门教程（一）](http://www.jianshu.com/p/a93c79e9f689)
[这可能是最好的RxJava 2.x 入门教程（二）](http://www.jianshu.com/p/b39afa92807e)
[这可能是最好的RxJava 2.x 入门教程（三）](http://www.jianshu.com/p/e9c79eacc8e3)
[这可能是最好的RxJava 2.x 入门教程（四）](http://www.jianshu.com/p/c08bfc58f4b6)
[这可能是最好的RxJava 2.x 入门教程（五）](http://www.jianshu.com/p/81fac37430dd)
GitHub 代码同步更新：[https://github.com/nanchen2251/RxJava2Examples](https://github.com/nanchen2251/RxJava2Examples)
为了满足大家的饥渴难耐，GitHub 将同步更新代码，主要包含基本的代码封装，RxJava 2.x 所有操作符应用场景介绍和实际应用场景，后期除了 RxJava 可能还会增添其他东西，总之，GitHub 上的 Demo 专为大家倾心打造。传送门：[https://github.com/nanchen2251/RxJava2Examples](https://github.com/nanchen2251/RxJava2Examples)

## 前言

RxJava 对大家而言肯定不陌生，其受欢迎程度不言而喻。而在去年的早些时候，官方便宣布，将在一段时间后不再对 RxJava 1.x 进行维护，而在仓库中另辟蹊径，开始对 RxJava 2.x 进行推广起来，我原本是不想写这么一套教程的，因为 RxJava 受欢迎度这么高，而且这 2.x 也出来了这么久，我坚信网上一定有很多超级大牛早已为大家避雷。然而很难过的是，我搜索了些时间，能搜出来的基本都是对 RxJava 1.x 的讲解，或者是 Blog 标题就没说清楚是否是 2.x 系列（对于我们这种标题党来说很难受）。这不，我就来抛砖引玉了。

咱们先不提别的，先为大家带点可能你早已熟知的干货——来自扔物线大神的[给Android开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)。

该文详细地为大家讲解了 RxJava 的优势、原理以及使用方式和适用情景，一定被众多的 Android 开发者视为神器。可惜，文章历史比较久远，基本都是讲解的 RxJava 1.x了。

那关注的小伙伴一定会问，那我没用过 RxJava 1.x ，还有必要先学习 1.x 的内容吗？

个人觉得不必要，因为 RxJava 2.x 是按照 **Reactive-Streams specification** 规范完全的重写的，完全独立于 RxJava 1.x 而存在，它改变了以往 RxJava 的用法。

额，由于个人能力水平有限，所以对于英文基础好的，大家可以去官网查阅相关 API 介绍，而对于英文不那么流畅的童鞋，我也为大家准备了干货：[RxJava2Examples (正在更新）](https://github.com/nanchen2251/RxJava2Examples)。

## 与RxJava 1.x的差异

 其实，我标题为入门教程，按理说应该从简单入门开始讲的，原谅我突然偏题了，因为我觉得可能大多数人都了解或者使用过RxJava 1.x（因为它真的太棒了）。虽然可能熟悉1.x 的你可以直接扒文档就可以了，但这么大的变化，请原谅我还在这里瞎比比。

- Nulls
  这是一个很大的变化，熟悉 RxJava 1.x 的童鞋一定都知道，1.x 是允许我们在发射事件的时候传入 null 值的，但现在我们的 2.x 不支持了，不信你试试？ 大大的 `NullPointerException` 教你做人。这意味着 `Observable<Void>` 不再发射任何值，而是正常结束或者抛出空指针。

- 2、Flowable 
  在 RxJava 1.x 中关于介绍 `backpressure` 部分有一个小小的遗憾，那就是没有用一个单独的类，而是使用 `Observable` 。而在 2.x 中 `Observable` 不支持背压了，将用一个全新的 Flowable 来支持背压。
  或许对于背压，有些小伙伴们还不是特别理解，这里简单说一下。大概就是指在异步场景中，被观察者发送事件的速度远快于观察者的处理速度的情况下，一种告诉上游的被观察者降低发送速度的策略。感兴趣的小伙伴可以模拟这种情况，在差距太大的时候，我们的内存会猛增，直到OOM。而我们的 Flowable 一定意义上可以解决这样的问题，但其实并不能完全解决，这个后面可能会提到。

- Single/Completable/Maybe
  其实这三者都差不多，`Single` 顾名思义，只能发送一个事件，和 `Observable `接受可变参数完全不同。而 `Completable` 侧重于观察结果，而 `Maybe` 是上面两种的结合体。也就是说，当你只想要某个事件的结果（true or false）的时候，你可以使用这种观察者模式。

- 线程调度相关
  这一块基本没什么改动，但细心的小伙伴一定会发现，RxJava 2.x 中已经没有了 `Schedulers.immediate()` 这个线程环境，还有 `Schedulers.test()`。

- Function相关
  熟悉 1.x 的小伙伴一定都知道，我们在1.x 中是有 `Func1`，`Func2`.....`FuncN`的，但 2.x 中将它们移除，而采用 `Function` 替换了 `Func1`，采用 `BiFunction` 替换了 `Func 2..N`。并且，它们都增加了 throws Exception，也就是说，妈妈再也不用担心我们做某些操作还需要 try-catch 了。

- 其他操作符相关
  如 `Func1...N` 的变化，现在同样用 `Consumer` 和 `BiConsumer` 对 `Action1` 和 `Action2` 进行了替换。后面的 `Action` 都被替换了，只保留了 `ActionN`。

## 附录
下面从官方截图展示 2.x 相对 1.x 的改动细节，仅供参考。
![](http://upload-images.jianshu.io/upload_images/3994917-14f7e368b8e0596b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/3994917-b447bbabccff5506.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/3994917-9863b5f713ac86d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/3994917-28ce3ee8a0ccdcf5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/3994917-548c743caff7c3e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/3994917-f20109ef808f04dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/3994917-93f5aee82d8fc8fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/3994917-4e8d8b566c245606.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/3994917-40ac7578f63fd10d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)做不完的开源，写不完的矫情。欢迎扫描下方二维码或者公众号搜索「nanchen」关注我的微信公众号，目前多运营 Android ，尽自己所能为你提升。如果你喜欢，为我点赞分享吧~
![nanchen](http://upload-images.jianshu.io/upload_images/3994917-3ec90e7c323323af?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)