这可能是最好的 RxJava 2.x 入门教程系列专栏
文章链接：
[这可能是最好的 RxJava 2.x 入门教程(完结版)](http://www.jianshu.com/p/0cd258eecf60)**【重磅推出】**
[这可能是最好的 RxJava 2.x 入门教程（一）](http://www.jianshu.com/p/a93c79e9f689)
[这可能是最好的 RxJava 2.x 入门教程（二）](http://www.jianshu.com/p/b39afa92807e)
[这可能是最好的 RxJava 2.x 入门教程（三）](http://www.jianshu.com/p/e9c79eacc8e3)
[这可能是最好的 RxJava 2.x 入门教程（四）](http://www.jianshu.com/p/c08bfc58f4b6)
[这可能是最好的 RxJava 2.x 入门教程（五）](http://www.jianshu.com/p/81fac37430dd)
GitHub 代码同步更新：[https://github.com/nanchen2251/RxJava2Examples](https://github.com/nanchen2251/RxJava2Examples)
为了满足大家的饥渴难耐，GitHub 将同步更新代码，主要包含基本的代码封装，RxJava 2.x 所有操作符应用场景介绍和实际应用场景，后期除了 RxJava 可能还会增添其他东西，总之，GitHub 上的 Demo 专为大家倾心打造。传送门：[https://github.com/nanchen2251/RxJava2Examples](https://github.com/nanchen2251/RxJava2Examples)

## 前言
年轻的老司机们，我这么勤的为大家分享，却少有催更的，好吧。其实写这个系列不是为了吸睛，那咱们继续写我们的 RxJava 2.x 的操作符。
## 正题
#### [distinct](http://reactivex.io/documentation/operators/distinct.html)
这个操作符非常的简单、通俗、易懂，就是简单的去重嘛，我甚至都不想贴代码，但人嘛，总得持之以恒。
![](http://upload-images.jianshu.io/upload_images/3994917-6b58db3710558ca6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```JAVA
 Observable.just(1, 1, 1, 2, 2, 3, 4, 5)
                .distinct()
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        mRxOperatorsText.append("distinct : " + integer + "\n");
                        Log.e(TAG, "distinct : " + integer + "\n");
                    }
                });
```

输出：
![](http://upload-images.jianshu.io/upload_images/3994917-9a9500abebd6097b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Log 日志显而易见，我们在经过 `dinstinct()` 后接收器接收到的事件只有1,2,3,4,5了。
#### [Filter](http://reactivex.io/documentation/operators/filter.html)
信我，`Filter` 你会很常用的，它的作用也很简单，过滤器嘛。可以接受一个参数，让其过滤掉不符合我们条件的值
![](http://upload-images.jianshu.io/upload_images/3994917-cbb8917000cd55fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
Observable.just(1, 20, 65, -5, 7, 19)
                .filter(new Predicate<Integer>() {
                    @Override
                    public boolean test(@NonNull Integer integer) throws Exception {
                        return integer >= 10;
                    }
                }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                mRxOperatorsText.append("filter : " + integer + "\n");
                Log.e(TAG, "filter : " + integer + "\n");
            }
        });
```

输出：
![](http://upload-images.jianshu.io/upload_images/3994917-c5cfda2812d122fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，我们过滤器舍去了小于 10 的值，所以最好的输出只有 20, 65, 19。

#### [buffer](http://reactivex.io/documentation/operators/buffer.html)
 `buffer` 操作符接受两个参数，`buffer(count,skip)`，作用是将 `Observable` 中的数据按 `skip` (步长) 分成最大不超过 count 的 `buffer` ，然后生成一个  `Observable` 。也许你还不太理解，我们可以通过我们的示例图和示例代码来进一步深化它。
![](http://upload-images.jianshu.io/upload_images/3994917-538728358e88a405.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
Observable.just(1, 2, 3, 4, 5)
                .buffer(3, 2)
                .subscribe(new Consumer<List<Integer>>() {
                    @Override
                    public void accept(@NonNull List<Integer> integers) throws Exception {
                        mRxOperatorsText.append("buffer size : " + integers.size() + "\n");
                        Log.e(TAG, "buffer size : " + integers.size() + "\n");
                        mRxOperatorsText.append("buffer value : ");
                        Log.e(TAG, "buffer value : " );
                        for (Integer i : integers) {
                            mRxOperatorsText.append(i + "");
                            Log.e(TAG, i + "");
                        }
                        mRxOperatorsText.append("\n");
                        Log.e(TAG, "\n");
                    }
                });
```
输出：
![](http://upload-images.jianshu.io/upload_images/3994917-08cdefce3e1e21ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如图，我们把 1, 2, 3, 4, 5 依次发射出来，经过 `buffer` 操作符，其中参数 `skip` 为 2， `count` 为 3，而我们的输出 依次是 123，345，5。显而易见，我们 `buffer` 的第一个参数是 `count`，代表最大取值，在事件足够的时候，一般都是取 `count` 个值，然后每次跳过 `skip` 个事件。其实看 Log 日志，我相信大家都明白了。

#### [timer](http://reactivex.io/documentation/operators/timer.html)
`timer` 很有意思，相当于一个定时任务。在 1.x 中它还可以执行间隔逻辑，但在 2.x 中此功能被交给了 `interval`，下一个会介绍。但需要注意的是，`timer`  和 `interval` 均默认在新线程。
![](http://upload-images.jianshu.io/upload_images/3994917-f29b7d6492d9f3f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
mRxOperatorsText.append("timer start : " + TimeUtil.getNowStrTime() + "\n");
        Log.e(TAG, "timer start : " + TimeUtil.getNowStrTime() + "\n");
        Observable.timer(2, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread()) // timer 默认在新线程，所以需要切换回主线程
                .subscribe(new Consumer<Long>() {
                    @Override
                    public void accept(@NonNull Long aLong) throws Exception {
                        mRxOperatorsText.append("timer :" + aLong + " at " + TimeUtil.getNowStrTime() + "\n");
                        Log.e(TAG, "timer :" + aLong + " at " + TimeUtil.getNowStrTime() + "\n");
                    }
                });
```

输出：
![](http://upload-images.jianshu.io/upload_images/3994917-77142cde60af8afc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
显而易见，当我们两次点击按钮触发这个事件的时候，接收被延迟了 2 秒。

#### [interval](http://reactivex.io/documentation/operators/interval.html)
如同我们上面可说，`interval` 操作符用于间隔时间执行某个操作，其接受三个参数，分别是第一次发送延迟，间隔时间，时间单位。
![](http://upload-images.jianshu.io/upload_images/3994917-c0cbc8f396617d7f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 ```java
mRxOperatorsText.append("interval start : " + TimeUtil.getNowStrTime() + "\n");
        Log.e(TAG, "interval start : " + TimeUtil.getNowStrTime() + "\n");
        Observable.interval(3,2, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread()) // 由于interval默认在新线程，所以我们应该切回主线程
                .subscribe(new Consumer<Long>() {
                    @Override
                    public void accept(@NonNull Long aLong) throws Exception {
                        mRxOperatorsText.append("interval :" + aLong + " at " + TimeUtil.getNowStrTime() + "\n");
                        Log.e(TAG, "interval :" + aLong + " at " + TimeUtil.getNowStrTime() + "\n");
                    }
                });
 ```
输出：
![](http://upload-images.jianshu.io/upload_images/3994917-908382027a946ab9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如同 Log 日志一样，第一次延迟了 3 秒后接收到，后面每次间隔了 2 秒。
然而，心细的小伙伴可能会发现，由于我们这个是间隔执行，所以当我们的Activity 都销毁的时候，实际上这个操作还依然在进行，所以，我们得花点小心思让我们在不需要它的时候干掉它。查看源码发现，我们subscribe(Cousumer<? super T> onNext)返回的是Disposable，我们可以在这上面做文章。
![](http://upload-images.jianshu.io/upload_images/3994917-aacb1daf93dc639f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 ```java
@Override
    protected void doSomething() {
        mRxOperatorsText.append("interval start : " + TimeUtil.getNowStrTime() + "\n");
        Log.e(TAG, "interval start : " + TimeUtil.getNowStrTime() + "\n");
        mDisposable = Observable.interval(3, 2, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread()) // 由于interval默认在新线程，所以我们应该切回主线程
                .subscribe(new Consumer<Long>() {
                    @Override
                    public void accept(@NonNull Long aLong) throws Exception {
                        mRxOperatorsText.append("interval :" + aLong + " at " + TimeUtil.getNowStrTime() + "\n");
                        Log.e(TAG, "interval :" + aLong + " at " + TimeUtil.getNowStrTime() + "\n");
                    }
                });
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (mDisposable != null && !mDisposable.isDisposed()) {
            mDisposable.dispose();
        }
    }
 ```

哈哈，再次验证，解决了我们的疑惑。

#### doOnNext
其实觉得 `doOnNext` 应该不算一个操作符，但考虑到其常用性，我们还是咬咬牙将它放在了这里。它的作用是让订阅者在接收到数据之前干点有意思的事情。假如我们在获取到数据之前想先保存一下它，无疑我们可以这样实现。
```java
Observable.just(1, 2, 3, 4)
                .doOnNext(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        mRxOperatorsText.append("doOnNext 保存 " + integer + "成功" + "\n");
                        Log.e(TAG, "doOnNext 保存 " + integer + "成功" + "\n");
                    }
                }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                mRxOperatorsText.append("doOnNext :" + integer + "\n");
                Log.e(TAG, "doOnNext :" + integer + "\n");
            }
        });
```
输出：
![](http://upload-images.jianshu.io/upload_images/3994917-f807f8be30b40e51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### [skip](http://reactivex.io/documentation/operators/skip.html)
 `skip` 很有意思，其实作用就和字面意思一样，接受一个 long 型参数 count ，代表跳过 count 个数目开始接收。
![](http://upload-images.jianshu.io/upload_images/3994917-27a77f4818941f33.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
Observable.just(1,2,3,4,5)
                .skip(2)
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        mRxOperatorsText.append("skip : "+integer + "\n");
                        Log.e(TAG, "skip : "+integer + "\n");
                    }
                });
```

输出：
![](http://upload-images.jianshu.io/upload_images/3994917-58b4338ddc42fc65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### [take](http://reactivex.io/documentation/operators/take.html)
`take`，接受一个 long 型参数 count ，代表至多接收 count 个数据。
![](http://upload-images.jianshu.io/upload_images/3994917-bd6d18a78013d0ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
Flowable.fromArray(1,2,3,4,5)
                .take(2)
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        mRxOperatorsText.append("take : "+integer + "\n");
                        Log.e(TAG, "accept: take : "+integer + "\n" );
                    }
                });
```

输出：
![](http://upload-images.jianshu.io/upload_images/3994917-cae2b5d836d9d181.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### [just](http://reactivex.io/documentation/operators/just.html)
` just`，没什么好说的，其实在前面各种例子都说明了，就是一个简单的发射器依次调用 `onNext()` 方法。
![](http://upload-images.jianshu.io/upload_images/3994917-6f7c4540d9a3b925.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
Observable.just("1", "2", "3")
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(@NonNull String s) throws Exception {
                        mRxOperatorsText.append("accept : onNext : " + s + "\n");
                        Log.e(TAG,"accept : onNext : " + s + "\n" );
                    }
                });
```

 输出：
![](http://upload-images.jianshu.io/upload_images/3994917-703a9c26b2939314.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 写在最后
好吧，本节先讲到这里，下节我们还是继续讲简单的操作符，虽然我们的教程比较枯燥，现在也不那么受人关注，但后面的系列我相信大家一定会非常喜欢的，我们下期再见！
​     代码全部同步到GitHub：[https://github.com/nanchen2251/RxJava2Examples](https://github.com/nanchen2251/RxJava2Examples)

![](http://upload-images.jianshu.io/upload_images/3994917-40ac7578f63fd10d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

做不完的开源，写不完的矫情。欢迎扫描下方二维码或者公众号搜索「nanchen」关注我的微信公众号，目前多运营 Android ，尽自己所能为你提升。如果你喜欢，为我点赞分享吧~
![nanchen](http://upload-images.jianshu.io/upload_images/3994917-3ec90e7c323323af?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)