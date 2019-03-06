这可能是最好的 RxJava 2.x 入门教程系列专栏
文章链接：
[这可能是最好的 RxJava 2.x 入门教程(完结版)](http://www.jianshu.com/p/0cd258eecf60)**[推荐直接看这个]**
[这可能是最好的RxJava 2.x 入门教程（一）](http://www.jianshu.com/p/a93c79e9f689)
[这可能是最好的RxJava 2.x 入门教程（二）](http://www.jianshu.com/p/b39afa92807e)
[这可能是最好的RxJava 2.x 入门教程（三）](http://www.jianshu.com/p/e9c79eacc8e3)
[这可能是最好的RxJava 2.x 入门教程（四）](http://www.jianshu.com/p/c08bfc58f4b6)
[这可能是最好的RxJava 2.x 入门教程（五）](http://www.jianshu.com/p/81fac37430dd)
GitHub 代码同步更新：[https://github.com/nanchen2251/RxJava2Examples](https://github.com/nanchen2251/RxJava2Examples)
为了满足大家的饥渴难耐，GitHub 将同步更新代码，主要包含基本的代码封装，RxJava 2.x 所有操作符应用场景介绍和实际应用场景，后期除了 RxJava 可能还会增添其他东西，总之，GitHub 上的 Demo 专为大家倾心打造。传送门：[https://github.com/nanchen2251/RxJava2Examples](https://github.com/nanchen2251/RxJava2Examples)

## 前言
很快我们就迎来了第二期，上一期我们主要讲解了 RxJava 1.x 到 2.x 的变化概览，相信各位熟练掌握RxJava 1.x的老司机们随便看一下变化概览就可以上手RxJava 2.x了，但为了满足更广大的年轻一代司机（未来也是老司机），在本节中，我们将学习RxJava 2.x 强大的操作符章节。
【注】以下所有操作符标题都可直接点击进入官方doc查看。

## 正题
#### [Create](http://reactivex.io/documentation/operators/create.html)

`create` 操作符应该是最常见的操作符了，主要用于产生一个 `Obserable` 被观察者对象，为了方便大家的认知，以后的教程中统一把被观察者 `Observable` 称为发射器（上游事件），观察者 `Observer` 称为接收器（下游事件）。
​     ![](http://upload-images.jianshu.io/upload_images/3994917-cfb570e933cf7abf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                mRxOperatorsText.append("Observable emit 1" + "\n");
                Log.e(TAG, "Observable emit 1" + "\n");
                e.onNext(1);
                mRxOperatorsText.append("Observable emit 2" + "\n");
                Log.e(TAG, "Observable emit 2" + "\n");
                e.onNext(2);
                mRxOperatorsText.append("Observable emit 3" + "\n");
                Log.e(TAG, "Observable emit 3" + "\n");
                e.onNext(3);
                e.onComplete();
                mRxOperatorsText.append("Observable emit 4" + "\n");
                Log.e(TAG, "Observable emit 4" + "\n" );
                e.onNext(4);
            }
        }).subscribe(new Observer<Integer>() {
            private int i;
            private Disposable mDisposable;

            @Override
            public void onSubscribe(@NonNull Disposable d) {
                mRxOperatorsText.append("onSubscribe : " + d.isDisposed() + "\n");
                Log.e(TAG, "onSubscribe : " + d.isDisposed() + "\n" );
                mDisposable = d;
            }

            @Override
            public void onNext(@NonNull Integer integer) {
                mRxOperatorsText.append("onNext : value : " + integer + "\n");
                Log.e(TAG, "onNext : value : " + integer + "\n" );
                i++;
                if (i == 2) {
                    // 在RxJava 2.x 中，新增的Disposable可以做到切断的操作，让Observer观察者不再接收上游事件
                    mDisposable.dispose();
                    mRxOperatorsText.append("onNext : isDisposable : " + mDisposable.isDisposed() + "\n");
                    Log.e(TAG, "onNext : isDisposable : " + mDisposable.isDisposed() + "\n");
                }
            }

            @Override
            public void onError(@NonNull Throwable e) {
                mRxOperatorsText.append("onError : value : " + e.getMessage() + "\n");
                Log.e(TAG, "onError : value : " + e.getMessage() + "\n" );
            }

            @Override
            public void onComplete() {
                mRxOperatorsText.append("onComplete" + "\n");
                Log.e(TAG, "onComplete" + "\n" );
            }
        });
```
输出：
![](http://upload-images.jianshu.io/upload_images/3994917-8980fc80715cf006.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

需要注意的几点是：
- 在发射事件中，我们在发射了数值 3 之后，直接调用了 `e.onComlete()`，虽然无法接收事件，但发送事件还是继续的。

- 另外一个值得注意的点是，在 RxJava 2.x 中，可以看到发射事件方法相比 1.x 多了一个 throws Excetion，意味着我们做一些特定操作再也不用 try-catch 了。

- 并且 2.x 中有一个 `Disposable` 概念，这个东西可以直接调用切断，可以看到，当它的 `isDisposed()` 返回为 false 的时候，接收器能正常接收事件，但当其为 true 的时候，接收器停止了接收。所以可以通过此参数动态控制接收事件了。

#### [Map](http://reactivex.io/documentation/operators/map.html)
`Map` 基本算是 RxJava 中一个最简单的操作符了，熟悉 RxJava 1.x 的知道，它的作用是对发射时间发送的每一个事件应用一个函数，是的每一个事件都按照指定的函数去变化，而在 2.x 中它的作用几乎一致。
![](http://upload-images.jianshu.io/upload_images/3994917-002d843b658b98e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onNext(3);
            }
        }).map(new Function<Integer, String>() {
            @Override
            public String apply(@NonNull Integer integer) throws Exception {
                return "This is result " + integer;
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(@NonNull String s) throws Exception {
                mRxOperatorsText.append("accept : " + s +"\n");
                Log.e(TAG, "accept : " + s +"\n" );
            }
        });
```
输出：
![](http://upload-images.jianshu.io/upload_images/3994917-c8409014dd7350e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

是的，`map` 基本作用就是将一个 `Observable` 通过某种函数关系，转换为另一种 `Observable`，上面例子中就是把我们的 `Integer` 数据变成了 `String` 类型。从Log日志显而易见。

#### [Zip](http://reactivex.io/documentation/operators/zip.html)
`zip` 专用于合并事件，该合并不是连接（连接操作符后面会说），而是两两配对，也就意味着，最终配对出的 `Observable` 发射事件数目只和少的那个相同。

```java
Observable.zip(getStringObservable(), getIntegerObservable(), new BiFunction<String, Integer, String>() {
            @Override
            public String apply(@NonNull String s, @NonNull Integer integer) throws Exception {
                return s + integer;
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(@NonNull String s) throws Exception {
                mRxOperatorsText.append("zip : accept : " + s + "\n");
                Log.e(TAG, "zip : accept : " + s + "\n");
            }
        });
```

```java
private Observable<String> getStringObservable() {
        return Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                if (!e.isDisposed()) {
                    e.onNext("A");
                    mRxOperatorsText.append("String emit : A \n");
                    Log.e(TAG, "String emit : A \n");
                    e.onNext("B");
                    mRxOperatorsText.append("String emit : B \n");
                    Log.e(TAG, "String emit : B \n");
                    e.onNext("C");
                    mRxOperatorsText.append("String emit : C \n");
                    Log.e(TAG, "String emit : C \n");
                }
            }
        });
    }

    private Observable<Integer> getIntegerObservable() {
        return Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                if (!e.isDisposed()) {
                    e.onNext(1);
                    mRxOperatorsText.append("Integer emit : 1 \n");
                    Log.e(TAG, "Integer emit : 1 \n");
                    e.onNext(2);
                    mRxOperatorsText.append("Integer emit : 2 \n");
                    Log.e(TAG, "Integer emit : 2 \n");
                    e.onNext(3);
                    mRxOperatorsText.append("Integer emit : 3 \n");
                    Log.e(TAG, "Integer emit : 3 \n");
                    e.onNext(4);
                    mRxOperatorsText.append("Integer emit : 4 \n");
                    Log.e(TAG, "Integer emit : 4 \n");
                    e.onNext(5);
                    mRxOperatorsText.append("Integer emit : 5 \n");
                    Log.e(TAG, "Integer emit : 5 \n");
                }
            }
        });
    }
```
输出：
![](http://upload-images.jianshu.io/upload_images/3994917-c6c28bda3583389b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

需要注意的是：
- `zip` 组合事件的过程就是分别从发射器 A 和发射器 B 各取出一个事件来组合，并且一个事件只能被使用一次，组合的顺序是严格按照事件发送的顺序来进行的，所以上面截图中，可以看到，1 永远是和 A 结合的，2 永远是和 B 结合的。

- 最终接收器收到的事件数量是和发送器发送事件最少的那个发送器的发送事件数目相同，所以如截图中，5 很孤单，没有人愿意和它交往，孤独终老的单身狗。

#### [Concat](http://reactivex.io/documentation/operators/concat.html)
对于单一的把两个发射器连接成一个发射器，虽然 `zip` 不能完成，但我们还是可以自力更生，官方提供的 `concat` 让我们的问题得到了完美解决。
![](http://upload-images.jianshu.io/upload_images/3994917-717b7a5bae136a0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
Observable.concat(Observable.just(1,2,3), Observable.just(4,5,6))
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        mRxOperatorsText.append("concat : "+ integer + "\n");
                        Log.e(TAG, "concat : "+ integer + "\n" );
                    }
                });
```
输出：
![](http://upload-images.jianshu.io/upload_images/3994917-f09630caa6b9e645.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如图，可以看到。发射器 B 把自己的三个孩子送给了发射器 A，让他们组合成了一个新的发射器，非常懂事的孩子，有条不紊的排序接收。

#### [FlatMap](http://reactivex.io/documentation/operators/flatmap.html)
`FlatMap` 是一个很有趣的东西，我坚信你在实际开发中会经常用到。它可以把一个发射器 `Observable` 通过某种方法转换为多个 `Observables`，然后再把这些分散的 `Observables`装进一个单一的发射器 `Observable`。但有个需要注意的是，`flatMap` 并不能保证事件的顺序，如果需要保证，需要用到我们下面要讲的 `ConcatMap`。

![](http://upload-images.jianshu.io/upload_images/3994917-ee85cd52a7cbb4f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 ```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onNext(3);
            }
        }).flatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(@NonNull Integer integer) throws Exception {
                List<String> list = new ArrayList<>();
                for (int i = 0; i < 3; i++) {
                    list.add("I am value " + integer);
                }
                int delayTime = (int) (1 + Math.random() * 10);
                return Observable.fromIterable(list).delay(delayTime, TimeUnit.MILLISECONDS);
            }
        }).subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(@NonNull String s) throws Exception {
                        Log.e(TAG, "flatMap : accept : " + s + "\n");
                        mRxOperatorsText.append("flatMap : accept : " + s + "\n");
                    }
                });
 ```

输出：
![](http://upload-images.jianshu.io/upload_images/3994917-24acfeb6e4f70ec4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一切都如我们预期中的有意思，为了区分 `concatMap`（下一个会讲），我在代码中特意动了一点小手脚，我采用一个随机数，生成一个时间，然后通过 `delay`（后面会讲）操作符，做一个小延时操作，而查看 Log 日志也确认验证了我们上面的说法，它是无序的。

#### [concatMap](http://reactivex.io/documentation/operators/flatmap.html)
上面其实就说了，`concatMap` 与 `FlatMap` 的唯一区别就是 `concatMap` 保证了顺序，所以，我们就直接把 `flatMap` 替换为 `concatMap` 验证吧。
```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onNext(3);
            }
        }).concatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(@NonNull Integer integer) throws Exception {
                List<String> list = new ArrayList<>();
                for (int i = 0; i < 3; i++) {
                    list.add("I am value " + integer);
                }
                int delayTime = (int) (1 + Math.random() * 10);
                return Observable.fromIterable(list).delay(delayTime, TimeUnit.MILLISECONDS);
            }
        }).subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(@NonNull String s) throws Exception {
                        Log.e(TAG, "flatMap : accept : " + s + "\n");
                        mRxOperatorsText.append("flatMap : accept : " + s + "\n");
                    }
                });
```

输出：
![](http://upload-images.jianshu.io/upload_images/3994917-f619ff85cd5199e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

结果的确和我们预想的一样。

## 写在最后

好了，这一节就先介绍到这里，下一节我们将学习其它的一些操作符，在操作符讲完后再带大家进入实际情景，希望持续关注，[代码传送门](https://github.com/nanchen2251/RxJava2Examples)

![](http://upload-images.jianshu.io/upload_images/3994917-40ac7578f63fd10d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

做不完的开源，写不完的矫情。欢迎扫描下方二维码或者公众号搜索「nanchen」关注我的微信公众号，目前多运营 Android ，尽自己所能为你提升。如果你喜欢，为我点赞分享吧~
![nanchen](http://upload-images.jianshu.io/upload_images/3994917-3ec90e7c323323af?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)