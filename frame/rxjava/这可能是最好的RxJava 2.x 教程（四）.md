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
最近很多小伙伴私信我，说自己很懊恼，对于 RxJava 2.x 系列一看就能明白，但自己写却又写不出来。如果 LZ 能放上实战情景教程就最好不过了。也是哈，单讲我们的操作符，也让我们的教程不温不火，但  LZ  自己选择的路，那跪着也要走完呀。所以，也就让我可怜的小伙伴们忍忍了，操作符马上就讲完了。

## 正题
#### Single
顾名思义，`Single` 只会接收一个参数，而 `SingleObserver` 只会调用 `onError()` 或者 `onSuccess()`。
```java
Single.just(new Random().nextInt())
                .subscribe(new SingleObserver<Integer>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {

                    }

                    @Override
                    public void onSuccess(@NonNull Integer integer) {
                        mRxOperatorsText.append("single : onSuccess : "+integer+"\n");
                        Log.e(TAG, "single : onSuccess : "+integer+"\n" );
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        mRxOperatorsText.append("single : onError : "+e.getMessage()+"\n");
                        Log.e(TAG, "single : onError : "+e.getMessage()+"\n");
                    }
                });
```
输出：
![](http://upload-images.jianshu.io/upload_images/3994917-d85e0e85a17dd7a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### [distinct](http://reactivex.io/documentation/operators/distinct.html)
去重操作符，简单的作用就是去重。
![](http://upload-images.jianshu.io/upload_images/3994917-8c146ae12e3f2186.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
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
![](http://upload-images.jianshu.io/upload_images/3994917-cf77b166d41ab325.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
很明显，发射器发送的事件，在接收的时候被去重了。

#### [debounce](http://reactivex.io/documentation/operators/debounce.html)
去除发送频率过快的项，看起来好像没啥用处，但你信我，后面绝对有地方很有用武之地。
![](http://upload-images.jianshu.io/upload_images/3994917-5409a530ac0e76b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> emitter) throws Exception {
                // send events with simulated time wait
                emitter.onNext(1); // skip
                Thread.sleep(400);
                emitter.onNext(2); // deliver
                Thread.sleep(505);
                emitter.onNext(3); // skip
                Thread.sleep(100);
                emitter.onNext(4); // deliver
                Thread.sleep(605);
                emitter.onNext(5); // deliver
                Thread.sleep(510);
                emitter.onComplete();
            }
        }).debounce(500, TimeUnit.MILLISECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        mRxOperatorsText.append("debounce :" + integer + "\n");
                        Log.e(TAG,"debounce :" + integer + "\n");
                    }
                });
```
输出：
![](http://upload-images.jianshu.io/upload_images/3994917-3c188c501ed07d59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
代码很清晰，去除发送间隔时间小于 500 毫秒的发射事件，所以 1 和 3 被去掉了。

#### [defer](http://reactivex.io/documentation/operators/defer.html)
简单地时候就是每次订阅都会创建一个新的 `Observable`，并且如果没有被订阅，就不会产生新的 `Observable`。
![](http://upload-images.jianshu.io/upload_images/3994917-0a5979b7b1266c69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
Observable<Integer> observable = Observable.defer(new Callable<ObservableSource<Integer>>() {
            @Override
            public ObservableSource<Integer> call() throws Exception {
                return Observable.just(1, 2, 3);
            }
        });


        observable.subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {

            }

            @Override
            public void onNext(@NonNull Integer integer) {
                mRxOperatorsText.append("defer : " + integer + "\n");
                Log.e(TAG, "defer : " + integer + "\n");
            }

            @Override
            public void onError(@NonNull Throwable e) {
                mRxOperatorsText.append("defer : onError : " + e.getMessage() + "\n");
                Log.e(TAG, "defer : onError : " + e.getMessage() + "\n");
            }

            @Override
            public void onComplete() {
                mRxOperatorsText.append("defer : onComplete\n");
                Log.e(TAG, "defer : onComplete\n");
            }
        });
```

输出：
![](http://upload-images.jianshu.io/upload_images/3994917-8355fe2e0b0bcef9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### [last](http://reactivex.io/documentation/operators/last.html)
`last` 操作符仅取出可观察到的最后一个值，或者是满足某些条件的最后一项。
![](http://upload-images.jianshu.io/upload_images/3994917-616bed35cca2755e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
Observable.just(1, 2, 3)
                .last(4)
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        mRxOperatorsText.append("last : " + integer + "\n");
                        Log.e(TAG, "last : " + integer + "\n");
                    }
                });
```

输出：
![](http://upload-images.jianshu.io/upload_images/3994917-d52cc90b0843b1d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### [merge](http://reactivex.io/documentation/operators/merge.html)
`merge` 顾名思义，熟悉版本控制工具的你一定不会不知道 merge 命令，而在 Rx 操作符中，`merge` 的作用是把多个 `Observable` 结合起来，接受可变参数，也支持迭代器集合。注意它和 `concat` 的区别在于，不用等到 发射器 A 发送完所有的事件再进行发射器 B 的发送。
![](http://upload-images.jianshu.io/upload_images/3994917-f2e0746a6e31aee1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
Observable.merge(Observable.just(1, 2), Observable.just(3, 4, 5))
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        mRxOperatorsText.append("merge :" + integer + "\n");
                        Log.e(TAG, "accept: merge :" + integer + "\n" );
                    }
                });
```

输出：
![](http://upload-images.jianshu.io/upload_images/3994917-8bc6e4686d8d5701.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### [reduce](http://reactivex.io/documentation/operators/reduce.html)
`reduce` 操作符每次用一个方法处理一个值，可以有一个 `seed` 作为初始值。
![](http://upload-images.jianshu.io/upload_images/3994917-d96ac93444a78c40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 ```java
Observable.just(1, 2, 3)
                .reduce(new BiFunction<Integer, Integer, Integer>() {
                    @Override
                    public Integer apply(@NonNull Integer integer, @NonNull Integer integer2) throws Exception {
                        return integer + integer2;
                    }
                }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                mRxOperatorsText.append("reduce : " + integer + "\n");
                Log.e(TAG, "accept: reduce : " + integer + "\n");
            }
        });
 ```
输出：
![](http://upload-images.jianshu.io/upload_images/3994917-0e76ccc3b95b6eee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，代码中，我们中间采用 reduce ，支持一个 function 为两数值相加，所以应该最后的值是：1 + 2 = 3 + 3 = 6 ， 而Log 日志完美解决了我们的问题。

#### [scan](http://reactivex.io/documentation/operators/scan.html)
`scan` 操作符作用和上面的 `reduce` 一致，唯一区别是 `reduce` 是个只追求结果的坏人，而 `scan` 会始终如一地把每一个步骤都输出。
![](http://upload-images.jianshu.io/upload_images/3994917-041e9e4f1b2e6355.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
Observable.just(1, 2, 3)
                .scan(new BiFunction<Integer, Integer, Integer>() {
                    @Override
                    public Integer apply(@NonNull Integer integer, @NonNull Integer integer2) throws Exception {
                        return integer + integer2;
                    }
                }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                mRxOperatorsText.append("scan " + integer + "\n");
                Log.e(TAG, "accept: scan " + integer + "\n");
            }
        });
```

输出：
![](http://upload-images.jianshu.io/upload_images/3994917-154573dd7cf2724a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看日志，没毛病。

#### [window](http://reactivex.io/documentation/operators/window.html)
按照实际划分窗口，将数据发送给不同的 `Observable`
![](http://upload-images.jianshu.io/upload_images/3994917-ae07467f8a135ceb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 ```java
mRxOperatorsText.append("window\n");
        Log.e(TAG, "window\n");
        Observable.interval(1, TimeUnit.SECONDS) // 间隔一秒发一次
                .take(15) // 最多接收15个
                .window(3, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Observable<Long>>() {
                    @Override
                    public void accept(@NonNull Observable<Long> longObservable) throws Exception {
                        mRxOperatorsText.append("Sub Divide begin...\n");
                        Log.e(TAG, "Sub Divide begin...\n");
                        longObservable.subscribeOn(Schedulers.io())
                                .observeOn(AndroidSchedulers.mainThread())
                                .subscribe(new Consumer<Long>() {
                                    @Override
                                    public void accept(@NonNull Long aLong) throws Exception {
                                        mRxOperatorsText.append("Next:" + aLong + "\n");
                                        Log.e(TAG, "Next:" + aLong + "\n");
                                    }
                                });
                    }
                });
 ```

输出：
![](http://upload-images.jianshu.io/upload_images/3994917-d660f4c3618a75f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 写在最后
至此，大部分 RxJava 2.x 的操作符就告一段落了，当然还有一些没有提到的操作符，不是说它们不重要，而是 LZ 也要考虑大家的情况，接下来就会根据实际应用场景来对 RxJava 2.x 发起冲锋。如果想看更多的数据，请移步 GitHub：[https://github.com/nanchen2251/RxJava2Examples](https://github.com/nanchen2251/RxJava2Examples)

![](http://upload-images.jianshu.io/upload_images/3994917-40ac7578f63fd10d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

做不完的开源，写不完的矫情。欢迎扫描下方二维码或者公众号搜索「nanchen」关注我的微信公众号，目前多运营 Android ，尽自己所能为你提升。如果你喜欢，为我点赞分享吧~
![nanchen](http://upload-images.jianshu.io/upload_images/3994917-3ec90e7c323323af?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)