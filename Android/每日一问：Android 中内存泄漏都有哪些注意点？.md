## 每日一问：Android 中内存泄漏都有哪些注意点？

内存泄漏对每一位 Android 开发一定是司空见惯，大家或多或少都肯定有些许接触。大家都知道，每一个手机都有一定的承载上限，多处的内存泄漏堆积一定会堆积如山，最终出现内存爆炸 OOM。

而这，也是极有可能在 Android 面试中一道常见的开放题。

内存泄漏的根本原因是**一个长生命周期的对象持有了一个短生命周期的对象。**如果你对垃圾回收机制有所了解，我想这个问题基本难不住你，因为知道了原理，自然不会去触碰这些极易导致内存泄漏的雷区。

> 该题重在积累，不需要死记硬背，自己多总结即可。

### 1. 长生命周期对象持有 Activity

这基本是最常见的内存泄漏了，比如

- 内部类形式使用 Handler 同时发送延时消息，或者在 Handler 里面执行耗时任务，在任务还没完成的时候 Activity 需要销毁。这时候由于 Handler 持有 Activity 的强引用导致 Activity 无法被回收。
- 同理内部类形式的使用 AsyncTask 执行耗时任务也会导致内存泄漏的发生。
- 单例作为最长生命周期的对象，自然不应该持有 Activity 从而导致内存泄漏发生；

针对上面这种情况，基本不必多说了，不要使用内部类或者匿名内部类做这样的处理就好了，实际上 IDE 也会弹出警告，我想大家应该还是都知道采用静态内部类或者在销毁页面的时候使用相关方法移除处理的。

> `Activity` 中匿名使用 `Handler`  实际上会导致 `Handler` 内部类持有外部类的引用，而 `SendMessage()` 的时候 `Message` 会持有 `Handler`，`enqueueMessage` 机制又会导致 `MeassageQueue` 持有 `Message`。所以当发送的是延迟消息那么 `Message` 并不会立即的遍历出来处理而是阻塞到对应的 `Message` 触发时间以后再处理。那么阻塞的这段时间中页面销毁一定会造成内存泄漏。

### 2. 各种注册操作没有对应的反注册

这一点基本不必多说，相信大家刚刚开始学习广播和 Service 的时候一定对此有所接触，然后就是比如我们常用的第三方框架 EventBus 也是一样的。平时使用的时候注意在对应的生命周期方法中进行反注册。

### 3. Bitmap 使用完没有注意 recycle()

Bitmap 作为大对象，在使用完毕一定要注意调用 `recycle()` 进行回收。`TypedArray` 、`Cursor`、各种流同理，一定要在最后调用自己的回收关闭方法处理。

### 4. WebView 使用不当

WebView 是非常常用的控件，但稍有不注意也会导致内存泄漏。内存泄漏的场景：　很多人使用 Webview 都喜欢采用布局引用方式, 这其实也是作为内存泄漏的一个隐患。当 Activity 被关闭时，Webview 不会被 GC 马上回收,而是提交给事务，进行队列处理，这样就造成了内存泄漏, 导致 Webview 无法及时回收。

目前所知的比较安全的方案是：

- 在布局中动态添加 WebView。
- 采用下面的方法。

```kotlin
override fun onDestroy() {
    webView?.apply {
        val parent = parent
        if (parent is ViewGroup) {
            parent.removeView(this)
        }
        stopLoading()
        // 退出时调用此方法，移除绑定的服务，否则某些特定系统会报错
        settings.javaScriptEnabled = false
        clearHistory()
        removeAllViews()
        destroy()
    }
}
```

### 5. 循环引用

循环引用导致内存泄漏比较少见，正常来讲不会有人写出 A 持有 B，B 持有 C，C 又持有A 这样的代码，不过总还是需要注意。

总的来说，内存泄漏很常见，但检测方式也很多。我们的 Android Studio 自带的 Monitors 就可以帮我们找到大部分内存问题，当然我们也可以采用譬如 LeakCanary 这样的库去做检测。

