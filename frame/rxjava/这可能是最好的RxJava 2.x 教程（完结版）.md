## 这可能是最好的RxJava 2.x 教程（完结版）

这可能是最好的 RxJava 2.x 入门教程系列专栏
文章链接：
[这可能是最好的RxJava 2.x 入门教程（一）](http://www.jianshu.com/p/a93c79e9f689)
[这可能是最好的RxJava 2.x 入门教程（二）](http://www.jianshu.com/p/b39afa92807e)
[这可能是最好的RxJava 2.x 入门教程（三）](http://www.jianshu.com/p/e9c79eacc8e3)
[这可能是最好的RxJava 2.x 入门教程（四）](http://www.jianshu.com/p/c08bfc58f4b6)
[这可能是最好的RxJava 2.x 入门教程（五）](http://www.jianshu.com/p/81fac37430dd)
GitHub 代码同步更新：[https://github.com/nanchen2251/RxJava2Examples](https://github.com/nanchen2251/RxJava2Examples)
为了满足大家的饥渴难耐，GitHub将同步更新代码，主要包含基本的代码封装，RxJava 2.x所有操作符应用场景介绍和实际应用场景，后期除了RxJava可能还会增添其他东西，总之，GitHub上的Demo专为大家倾心打造。传送门：[https://github.com/nanchen2251/RxJava2Examples](https://github.com/nanchen2251/RxJava2Examples)

## 为什么要学 RxJava？
提升开发效率，降低维护成本一直是开发团队永恒不变的宗旨。近两年来国内的技术圈子中越来越多的开始提及 [RxJava](https://github.com/ReactiveX/RxJava) ，越来越多的应用和面试中都会有 [RxJava](https://github.com/ReactiveX/RxJava) ，而就目前的情况，Android 的网络库基本被 [Retrofit](https://github.com/square/retrofit) + [OkHttp](https://github.com/square/okhttp) 一统天下了，而配合上**响应式编程** [RxJava](https://github.com/ReactiveX/RxJava) 可谓如鱼得水。想必大家肯定被近期的 [Kotlin](https://github.com/JetBrains/kotlin) 炸开了锅，笔者也在闲暇之时去了解了一番（作为一个与时俱进的有理想的青年怎么可能不与时俱进？），发现其中有个非常好的优点就是简洁，支持函数式编程。是的， RxJava 最大的优点也是简洁，但它不止是简洁，而且是** 随着程序逻辑变得越来越复杂，它依然能够保持简洁 **（这货洁身自好呀有木有）。
![](http://upload-images.jianshu.io/upload_images/3994917-da982dc6fb602ee2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
咳咳，要例子，猛戳这里：[给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083#toc_2)

## 什么是响应式编程
上面我们提及了**响应式编程**，不少新司机对它可谓一脸懵逼，那什么是**响应式编程**呢？**响应式编程**是一种基于异步数据流概念的编程模式。数据流就像一条河：它可以被观测，被过滤，被操作，或者为新的消费者与另外一条流合并为一条新的流。

响应式编程的一个关键概念是事件。事件可以被等待，可以触发过程，也可以触发其它事件。事件是唯一的以合适的方式将我们的现实世界映射到我们的软件中：如果屋里太热了我们就打开一扇窗户。同样的，当我们的天气app从服务端获取到新的天气数据后，我们需要更新app上展示天气信息的UI；汽车上的车道偏移系统探测到车辆偏移了正常路线就会提醒驾驶者纠正，就是是响应事件。

今天，响应式编程最通用的一个场景是UI：我们的移动App必须做出对网络调用、用户触摸输入和系统弹框的响应。在这个世界上，软件之所以是事件驱动并响应的是因为现实生活也是如此。


## 为什么出了一个系列后还有完结版？
[RxJava](https://github.com/ReactiveX/RxJava) 这些年可谓越来越流行，而在去年的晚些时候发布了2.0正式版。大半年已过，虽然网上已经出现了大部分的 RxJava 教程（其实细心的你还是会发现 1.x 的超级多），前些日子，笔者花了大约两周的闲暇之时写了 RxJava 2.x 系列教程，也得到了不少反馈，其中就有不少读者觉得每一篇的教程太短，抑或是希望更多的侧重适用场景的介绍，在这样的大前提下，这篇完结版教程就此诞生，仅供各位新司机采纳。
![](http://upload-images.jianshu.io/upload_images/3994917-f60bdfe3afcba051.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 开始
RxJava 2.x 已经按照 Reactive-Streams specification 规范完全的重写了，maven也被放在了` io.reactivex.rxjava2:rxjava:2.x.y` 下，所以 RxJava 2.x 独立于 RxJava 1.x 而存在，而随后官方宣布的将在一段时间后终止对 RxJava 1.x 的维护，所以对于熟悉 RxJava 1.x 的老司机自然可以直接看一下 [2.x 的文档](http://reactivex.io/RxJava/2.x/javadoc/)和异同就能轻松上手了，而对于不熟悉的年轻司机，不要慌，本酱带你装逼带你飞，马上就发车，坐稳了：https://github.com/nanchen2251/RxJava2Examples

你只需要在 build.gradle 中加上：`compile 'io.reactivex.rxjava2:rxjava:2.1.1'`（2.1.1为写此文章时的最新版本）
![](http://upload-images.jianshu.io/upload_images/3994917-ea91d3468b5063d3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 接口变化
RxJava 2.x 拥有了新的特性，其依赖于4个基础接口，它们分别是
- Publisher
- Subscriber
- Subscription
- Processor

其中最核心的莫过于 ` Publisher` 和 `Subscriber`。`Publisher` 可以发出一系列的事件，而 ` Subscriber ` 负责和处理这些事件。

其中用的比较多的自然是 `Publisher` 的 `Flowable`，它支持背压。关于背压给个简洁的定义就是：
> 背压是指在异步场景中，被观察者发送事件速度远快于观察者的处理速度的情况下，一种告诉上游的被观察者降低发送速度的策略。

简而言之，**背压是流速控制的一种策略**。有兴趣的可以看一下[官方对于背压的讲解](https://github.com/ReactiveX/RxJava/wiki/Backpressure)。

可以明显地发现，RxJava 2.x 最大的改动就是对于 `backpressure` 的处理，为此将原来的 `Observable` 拆分成了新的 `Observable` 和 `Flowable`，同时其他相关部分也同时进行了拆分，但令人庆幸的是，是它，是它，还是它，还是我们最熟悉和最喜欢的 RxJava。

## 观察者模式
大家可能都知道， **RxJava 以观察者模式为骨架，在 2.0 中依旧如此**。

不过此次更新中，出现了两种观察者模式：

- Observable ( 被观察者 ) / Observer ( 观察者 )
- Flowable （被观察者）/ Subscriber （观察者）
  ![](http://upload-images.jianshu.io/upload_images/3994917-21e4dcc1b5e3196a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 RxJava 2.x 中，Observable 用于订阅 Observer，不再支持背压（1.x 中可以使用背压策略），而 Flowable 用于订阅 Subscriber ， 是支持背压（Backpressure）的。

## Observable
在 RxJava 1.x 中，我们最熟悉的莫过于 `Observable` 这个类了，笔者在刚刚使用 RxJava 2.x 的时候，创建了 一个 `Observable`，瞬间一脸懵逼有木有，居然连我们最最熟悉的 `Subscriber` 都没了，取而代之的是 `ObservableEmmiter`，俗称发射器。此外，由于没有了`Subscriber`的踪影，我们创建观察者时需使用 `Observer`。而 `Observer` 也不是我们熟悉的那个 `Observer`，又出现了一个 `Disposable` 参数带你装逼带你飞。

废话不多说，从会用开始，还记得 RxJava 的三部曲吗？
![](http://upload-images.jianshu.io/upload_images/1167421-58e89ad932b6a714.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
** 第一步：初始化 Observable **
** 第二步：初始化 Observer **
** 第三步：建立订阅关系 **
![](http://upload-images.jianshu.io/upload_images/3994917-cd3c64eb6fa97663.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
Observable.create(new ObservableOnSubscribe<Integer>() { // 第一步：初始化Observable
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                Log.e(TAG, "Observable emit 1" + "\n");
                e.onNext(1);
                Log.e(TAG, "Observable emit 2" + "\n");
                e.onNext(2);
                Log.e(TAG, "Observable emit 3" + "\n");
                e.onNext(3);
                e.onComplete();
                Log.e(TAG, "Observable emit 4" + "\n" );
                e.onNext(4);
            }
        }).subscribe(new Observer<Integer>() { // 第三步：订阅

            // 第二步：初始化Observer
            private int i;
            private Disposable mDisposable;

            @Override
            public void onSubscribe(@NonNull Disposable d) {      
                mDisposable = d;
            }

            @Override
            public void onNext(@NonNull Integer integer) {
                i++;
                if (i == 2) {
                    // 在RxJava 2.x 中，新增的Disposable可以做到切断的操作，让Observer观察者不再接收上游事件
                    mDisposable.dispose();
                }
            }

            @Override
            public void onError(@NonNull Throwable e) {
                Log.e(TAG, "onError : value : " + e.getMessage() + "\n" );
            }

            @Override
            public void onComplete() {
                Log.e(TAG, "onComplete" + "\n" );
            }
        });
```

不难看出，RxJava 2.x 与 1.x 还是存在着一些区别的。首先，创建 `Observable` 时，回调的是 `ObservableEmitter` ，字面意思即发射器，并且直接 throws Exception。其次，在创建的 Observer 中，也多了一个回调方法：`onSubscribe`，传递参数为`Disposable`，`Disposable` 相当于 RxJava 1.x 中的 `Subscription `， 用于解除订阅。可以看到示例代码中，在  i 自增到 2 的时候，订阅关系被切断。
```java
07-03 14:24:11.663 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: onSubscribe : false
07-03 14:24:11.664 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: Observable emit 1
07-03 14:24:11.665 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: onNext : value : 1
07-03 14:24:11.666 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: Observable emit 2
07-03 14:24:11.667 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: onNext : value : 2
07-03 14:24:11.668 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: onNext : isDisposable : true
07-03 14:24:11.669 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: Observable emit 3
07-03 14:24:11.670 18467-18467/com.nanchen.rxjava2examples E/RxCreateActivity: Observable emit 4
```
当然，我们的 RxJava 2.x 也为我们保留了简化订阅方法，我们可以根据需求，进行相应的简化订阅，只不过传入对象改为了 `Consumer`。

`Consumer` 即消费者，用于接收单个值，`BiConsumer` 则是接收两个值，`Function` 用于变换对象，`Predicate` 用于判断。这些接口命名大多参照了 Java 8 ，熟悉 Java 8 新特性的应该都知道意思，这里也不再赘述。

## 线程调度
关于线程切换这点，RxJava 1.x 和 RxJava 2.x 的实现思路是一样的。这里简单的说一下，以便于我们的新司机入手。
#### subScribeOn
同 RxJava 1.x 一样，`subscribeOn` 用于指定 `subscribe()` 时所发生的线程，从源码角度可以看出，内部线程调度是通过 `ObservableSubscribeOn`来实现的。
```java
    @SchedulerSupport(SchedulerSupport.CUSTOM)
    public final Observable<T> subscribeOn(Scheduler scheduler) {
        ObjectHelper.requireNonNull(scheduler, "scheduler is null");
        return RxJavaPlugins.onAssembly(new ObservableSubscribeOn<T>(this, scheduler));
    }
```
`ObservableSubscribeOn` 的核心源码在 `subscribeActual` 方法中，通过代理的方式使用 `SubscribeOnObserver` 包装 `Observer` 后，设置 `Disposable` 来将 ` subscribe` 切换到 `Scheduler` 线程中。
#### observeOn
`observeOn` 方法用于指定下游 `Observer` 回调发生的线程。
```java
   @SchedulerSupport(SchedulerSupport.CUSTOM)
    public final Observable<T> observeOn(Scheduler scheduler, boolean delayError, int bufferSize) {
        ObjectHelper.requireNonNull(scheduler, "scheduler is null");
        ObjectHelper.verifyPositive(bufferSize, "bufferSize");
        return RxJavaPlugins.onAssembly(new ObservableObserveOn<T>(this, scheduler, delayError, bufferSize));
    }
```
#### 线程切换需要注意的
RxJava 内置的线程调度器的确可以让我们的线程切换得心应手，但其中也有些需要注意的地方。
- 简单地说，`subscribeOn()` 指定的就是发射事件的线程，`observerOn` 指定的就是订阅者接收事件的线程。
- 多次指定发射事件的线程只有第一次指定的有效，也就是说多次调用 `subscribeOn()` 只有第一次的有效，其余的会被忽略。
- 但多次指定订阅者接收线程是可以的，也就是说每调用一次 `observerOn()`，下游的线程就会切换一次。

![](http://upload-images.jianshu.io/upload_images/3994917-398ee3ef9fcacfbd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                Log.e(TAG, "Observable thread is : " + Thread.currentThread().getName());
                e.onNext(1);
                e.onComplete();
            }
        }).subscribeOn(Schedulers.newThread())
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .doOnNext(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        Log.e(TAG, "After observeOn(mainThread)，Current thread is " + Thread.currentThread().getName());
                    }
                })
                .observeOn(Schedulers.io())
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        Log.e(TAG, "After observeOn(io)，Current thread is " + Thread.currentThread().getName());
                    }
                });
```
输出：
```java
07-03 14:54:01.177 15121-15438/com.nanchen.rxjava2examples E/RxThreadActivity: Observable thread is : RxNewThreadScheduler-1
07-03 14:54:01.178 15121-15121/com.nanchen.rxjava2examples E/RxThreadActivity: After observeOn(mainThread)，Current thread is main
07-03 14:54:01.179 15121-15439/com.nanchen.rxjava2examples E/RxThreadActivity: After observeOn(io)，Current thread is RxCachedThreadScheduler-2
```
实例代码中，分别用 `Schedulers.newThread()` 和 `Schedulers.io()` 对发射线程进行切换，并采用 `observeOn(AndroidSchedulers.mainThread()` 和 `Schedulers.io()` 进行了接收线程的切换。可以看到输出中发射线程仅仅响应了第一个 `newThread`，但每调用一次 `observeOn()` ，线程便会切换一次，因此如果我们有类似的需求时，便知道如何处理了。

RxJava 中，已经内置了很多线程选项供我们选择，例如有：
- `Schedulers.io()` 代表io操作的线程, 通常用于网络,读写文件等io密集型的操作；
- `Schedulers.computation()` 代表CPU计算密集型的操作, 例如需要大量计算的操作；
- `Schedulers.newThread()` 代表一个常规的新线程；
- `AndroidSchedulers.mainThread()` 代表Android的主线程

这些内置的 `Scheduler` 已经足够满足我们开发的需求，因此我们应该使用内置的这些选项，而 RxJava 内部使用的是线程池来维护这些线程，所以效率也比较高。

## 操作符
关于操作符，在[官方文档](http://reactivex.io/documentation/operators.html)中已经做了非常完善的讲解，并且笔者[前面的系列教程](http://www.jianshu.com/p/b39afa92807e)中也着重讲解了绝大多数的操作符作用，这里受于篇幅限制，就不多做赘述，只挑选几个进行实际情景的讲解。
![](http://upload-images.jianshu.io/upload_images/3994917-4b63ed68c78214ce.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### map

![](http://upload-images.jianshu.io/upload_images/3994917-002d843b658b98e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`map` 操作符可以将一个 `Observable` 对象通过某种关系转换为另一个`Observable` 对象。在 2.x 中和 1.x 中作用几乎一致，不同点在于：2.x 将 1.x 中的 `Func1` 和 `Func2` 改为了 `Function` 和 `BiFunction`。

#### 采用 map 操作符进行网络数据解析
想必大家都知道，很多时候我们在使用 RxJava 的时候总是和 Retrofit 进行结合使用，而为了方便演示，这里我们就暂且采用 OkHttp3 进行演示，配合 `map`，`doOnNext` ，线程切换进行简单的网络请求：
1）通过 Observable.create() 方法，调用 OkHttp 网络请求；
2）通过 map 操作符集合 gson，将 Response 转换为 bean 类；
3）通过 doOnNext() 方法，解析 bean 中的数据，并进行数据库存储等操作；
4）调度线程，在子线程中进行耗时操作任务，在主线程中更新 UI ；
5）通过 subscribe()，根据请求成功或者失败来更新 UI 。
```java
Observable.create(new ObservableOnSubscribe<Response>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Response> e) throws Exception {
                Builder builder = new Builder()
                        .url("http://api.avatardata.cn/MobilePlace/LookUp?key=ec47b85086be4dc8b5d941f5abd37a4e&mobileNumber=13021671512")
                        .get();
                Request request = builder.build();
                Call call = new OkHttpClient().newCall(request);
                Response response = call.execute();
                e.onNext(response);
            }
        }).map(new Function<Response, MobileAddress>() {
                    @Override
                    public MobileAddress apply(@NonNull Response response) throws Exception {
                        if (response.isSuccessful()) {
                            ResponseBody body = response.body();
                            if (body != null) {
                                Log.e(TAG, "map:转换前:" + response.body());
                                return new Gson().fromJson(body.string(), MobileAddress.class);
                            }
                        }
                        return null;
                    }
                }).observeOn(AndroidSchedulers.mainThread())
                .doOnNext(new Consumer<MobileAddress>() {
                    @Override
                    public void accept(@NonNull MobileAddress s) throws Exception {
                        Log.e(TAG, "doOnNext: 保存成功：" + s.toString() + "\n");
                    }
                }).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<MobileAddress>() {
                    @Override
                    public void accept(@NonNull MobileAddress data) throws Exception {
                        Log.e(TAG, "成功:" + data.toString() + "\n");
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(@NonNull Throwable throwable) throws Exception {
                        Log.e(TAG, "失败：" + throwable.getMessage() + "\n");
                    }
                });
```

#### concat
![](http://upload-images.jianshu.io/upload_images/3994917-717b7a5bae136a0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
`concat` 可以做到不交错的发射两个甚至多个 Observable 的发射事件，并且只有前一个 `Observable` 终止(`onComplete`) 后才会订阅下一个 `Observable`。

#### 采用 concat 操作符先读取缓存再通过网络请求获取数据
想必在实际应用中，很多时候（对数据操作不敏感时）都需要我们先读取缓存的数据，如果缓存没有数据，再通过网络请求获取，随后在主线程更新我们的UI。

`concat` 操作符简直就是为我们这种需求量身定做。

利用 `concat` 的必须调用 `onComplete` 后才能订阅下一个 `Observable` 的特性，我们就可以先读取缓存数据，倘若获取到的缓存数据不是我们想要的，再调用 `onComplete()` 以执行获取网络数据的 `Observable`，如果缓存数据能应我们所需，则直接调用 `onNext()`，防止过度的网络请求，浪费用户的流量。

```java
Observable<FoodList> cache = Observable.create(new ObservableOnSubscribe<FoodList>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<FoodList> e) throws Exception {
                Log.e(TAG, "create当前线程:"+Thread.currentThread().getName() );
                FoodList data = CacheManager.getInstance().getFoodListData();

                // 在操作符 concat 中，只有调用 onComplete 之后才会执行下一个 Observable
                if (data != null){ // 如果缓存数据不为空，则直接读取缓存数据，而不读取网络数据
                    isFromNet = false;
                    Log.e(TAG, "\nsubscribe: 读取缓存数据:" );
                    runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            mRxOperatorsText.append("\nsubscribe: 读取缓存数据:\n");
                        }
                    });

                    e.onNext(data);
                }else {
                    isFromNet = true;
                    runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            mRxOperatorsText.append("\nsubscribe: 读取网络数据:\n");
                        }
                    });
                    Log.e(TAG, "\nsubscribe: 读取网络数据:" );
                    e.onComplete();
                }


            }
        });

        Observable<FoodList> network = Rx2AndroidNetworking.get("http://www.tngou.net/api/food/list")
                .addQueryParameter("rows",10+"")
                .build()
                .getObjectObservable(FoodList.class);


        // 两个 Observable 的泛型应当保持一致

        Observable.concat(cache,network)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<FoodList>() {
                    @Override
                    public void accept(@NonNull FoodList tngouBeen) throws Exception {
                        Log.e(TAG, "subscribe 成功:"+Thread.currentThread().getName() );
                        if (isFromNet){
                            mRxOperatorsText.append("accept : 网络获取数据设置缓存: \n");
                            Log.e(TAG, "accept : 网络获取数据设置缓存: \n"+tngouBeen.toString() );
                            CacheManager.getInstance().setFoodListData(tngouBeen);
                        }

                        mRxOperatorsText.append("accept: 读取数据成功:" + tngouBeen.toString()+"\n");
                        Log.e(TAG, "accept: 读取数据成功:" + tngouBeen.toString());
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(@NonNull Throwable throwable) throws Exception {
                        Log.e(TAG, "subscribe 失败:"+Thread.currentThread().getName() );
                        Log.e(TAG, "accept: 读取数据失败："+throwable.getMessage() );
                        mRxOperatorsText.append("accept: 读取数据失败："+throwable.getMessage()+"\n");
                    }
                });
```
有时候我们的缓存可能还会分为 memory 和 disk ，实际上都差不多，无非是多写点 `Observable` ，然后通过 `concat` 合并即可。

#### flatMap 实现多个网络请求依次依赖
想必这种情况也在实际情况中比比皆是，例如用户注册成功后需要自动登录，我们只需要先通过注册接口注册用户信息，注册成功后马上调用登录接口进行自动登录即可。

我们的 `flatMap` 恰好解决了这种应用场景，`flatMap` 操作符可以将一个发射数据的 `Observable` 变换为多个 `Observables` ，然后将它们发射的数据合并后放到一个单独的 `Observable`，利用这个特性，我们很轻松地达到了我们的需求。

```java
Rx2AndroidNetworking.get("http://www.tngou.net/api/food/list")
                .addQueryParameter("rows", 1 + "")
                .build()
                .getObjectObservable(FoodList.class) // 发起获取食品列表的请求，并解析到FootList
                .subscribeOn(Schedulers.io())        // 在io线程进行网络请求
                .observeOn(AndroidSchedulers.mainThread()) // 在主线程处理获取食品列表的请求结果
                .doOnNext(new Consumer<FoodList>() {
                    @Override
                    public void accept(@NonNull FoodList foodList) throws Exception {
                        // 先根据获取食品列表的响应结果做一些操作
                        Log.e(TAG, "accept: doOnNext :" + foodList.toString());
                        mRxOperatorsText.append("accept: doOnNext :" + foodList.toString()+"\n");
                    }
                })
                .observeOn(Schedulers.io()) // 回到 io 线程去处理获取食品详情的请求
                .flatMap(new Function<FoodList, ObservableSource<FoodDetail>>() {
                    @Override
                    public ObservableSource<FoodDetail> apply(@NonNull FoodList foodList) throws Exception {
                        if (foodList != null && foodList.getTngou() != null && foodList.getTngou().size() > 0) {
                            return Rx2AndroidNetworking.post("http://www.tngou.net/api/food/show")
                                    .addBodyParameter("id", foodList.getTngou().get(0).getId() + "")
                                    .build()
                                    .getObjectObservable(FoodDetail.class);
                        }
                        return null;

                    }
                })
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<FoodDetail>() {
                    @Override
                    public void accept(@NonNull FoodDetail foodDetail) throws Exception {
                        Log.e(TAG, "accept: success ：" + foodDetail.toString());
                        mRxOperatorsText.append("accept: success ：" + foodDetail.toString()+"\n");
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(@NonNull Throwable throwable) throws Exception {
                        Log.e(TAG, "accept: error :" + throwable.getMessage());
                        mRxOperatorsText.append("accept: error :" + throwable.getMessage()+"\n");
                    }
                });
```

#### 善用 zip 操作符，实现多个接口数据共同更新 UI 
在实际应用中，我们极有可能会在一个页面显示的数据来源于多个接口，这时候我们的 `zip` 操作符为我们排忧解难。

`zip` 操作符可以将多个 `Observable` 的数据结合为一个数据源再发射出去。

```java
Observable<MobileAddress> observable1 = Rx2AndroidNetworking.get("http://api.avatardata.cn/MobilePlace/LookUp?key=ec47b85086be4dc8b5d941f5abd37a4e&mobileNumber=13021671512")
                .build()
                .getObjectObservable(MobileAddress.class);

        Observable<CategoryResult> observable2 = Network.getGankApi()
                .getCategoryData("Android",1,1);

        Observable.zip(observable1, observable2, new BiFunction<MobileAddress, CategoryResult, String>() {
            @Override
            public String apply(@NonNull MobileAddress mobileAddress, @NonNull CategoryResult categoryResult) throws Exception {
                return "合并后的数据为：手机归属地："+mobileAddress.getResult().getMobilearea()+"人名："+categoryResult.results.get(0).who;
            }
        }).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(@NonNull String s) throws Exception {
                        Log.e(TAG, "accept: 成功：" + s+"\n");
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(@NonNull Throwable throwable) throws Exception {
                        Log.e(TAG, "accept: 失败：" + throwable+"\n");
                    }
                });
```

#### 采用 interval 操作符实现心跳间隔任务
想必即时通讯等需要轮训的任务在如今的 APP 中已是很常见，而 RxJava 2.x 的 `interval` 操作符可谓完美地解决了我们的疑惑。

这里就简单的意思一下轮训。

```java 
private Disposable mDisposable;
    @Override
    protected void doSomething() {
        mDisposable = Flowable.interval(1, TimeUnit.SECONDS)
                .doOnNext(new Consumer<Long>() {
                    @Override
                    public void accept(@NonNull Long aLong) throws Exception {
                        Log.e(TAG, "accept: doOnNext : "+aLong );
                    }
                })
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Long>() {
                    @Override
                    public void accept(@NonNull Long aLong) throws Exception {
                        Log.e(TAG, "accept: 设置文本 ："+aLong );
                        mRxOperatorsText.append("accept: 设置文本 ："+aLong +"\n");
                    }
                });
    }

    /**
     * 销毁时停止心跳
     */
    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (mDisposable != null){
            mDisposable.dispose();
        }
    }
```

## RxJava 1.x 如何平滑升级到 RxJava 2.x？
由于 RxJava 2.x 变化较大无法直接升级，幸运的是，官方为我们提供了 [RxJava2Interrop]([https://github.com/akarnokd/RxJava2Interop](https://github.com/akarnokd/RxJava2Interop)) 这个库，可以方便地把 RxJava 1.x 升级到 RxJava 2.x，或者将 RxJava 2.x 转回到 RxJava 1.x。

## 写在最后
本酱看你都看到这儿了，实为未来的栋梁之才，所以且送你一本经书：
[https://github.com/nanchen2251/RxJava2Examples](https://github.com/nanchen2251/RxJava2Examples)

![](http://upload-images.jianshu.io/upload_images/3994917-b471448aeac2bb61.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![GIF.gif](http://upload-images.jianshu.io/upload_images/3994917-5f5caa087ed54c0d.gif?imageMogr2/auto-orient/strip)

![](http://upload-images.jianshu.io/upload_images/3994917-40ac7578f63fd10d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**做不完的开源，写不完的矫情。欢迎扫描下方二维码或者公众号搜索「nanchen」关注我的微信公众号，目前多运营 Android ，尽自己所能为你提升。现在还有纯面试系列哟~涵盖 Java、数据结构与算法、计算机网络、Android 基础。如果你喜欢，为我点赞分享吧~**
![nanchen](http://upload-images.jianshu.io/upload_images/3994917-3ec90e7c323323af?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)