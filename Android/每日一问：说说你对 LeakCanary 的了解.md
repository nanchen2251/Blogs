## 每日一问：说说你对 LeakCanary 的了解

昨天的问题说到了关于 [内存泄漏需要注意的点](<https://www.jianshu.com/p/b0345cb39819>)，在文章最后有说到 [LeakCanary](<https://github.com/square/leakcanary>) 检测内存泄漏。实际上，我相信绝大多数人也知道甚至使用过这个库。

> 这个系列通常来说如果发现了不错的资源，会选择直接截取部分拿过来，所以对于文章底部的参考链接一般都是非常不错的，可以直接去看哟~

#### LeakCanary 的基本工作流程是怎样的？

LeakCanary 的使用方式非常简单，只需要在 build.gradle 里面直接写上依赖，并且在 Application 类里面做注册就可以了。

> 当然，需要在 Application 里面注册这样的操作仅在大多数人接触的 1.x 版本，实际上 LeakCanary 现在已经升级到了 2.x 版本，代码侵入性更低，而且纯 Kotlin 写法。从 Google 各种 Demo 主推 Kotlin 以及各种主流库都在使用 Kotlin 编写来看可见 Kotlin 确实在 Android 开发中愈发重要，没使用的小伙伴必须得去学习一波了，目前我也是纯 Kotlin 做开发的。

对于工作原理我相信大家应该也是或多或少有一定了解，这里刚好有一张非常不错的流程图就直接借用过来了，另外他从源码角度理解 LeakCanary 的这篇文章也写的非常不错，感兴趣的点击文章底部的链接直达。

![](https://upload-images.jianshu.io/upload_images/6544890-9a37807f59f71b55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/982/format/webp)

#### 初次使用 LeakCanary 为什么没有 Icon 入口

我们常常在使用 LeakCanary 的时候会发现这样一个问题：最开始并没有出现 LeakCanary 的 Launcher icon，但当出现了内存泄漏警告的时候系统桌面就多了这么一个图标，一般情况下都是会非常好奇的。

从 1.x 的源码中就可以看出端倪。在 leakcanary-android 的 manifast 中，我们可以看到相关配置：

```xml
<!--leakcanary-sample/src/main/AndroidManifest.xml-->
<service
    android:name=".internal.HeapAnalyzerService"
    android:process=":leakcanary"
    android:enabled="false"
    />
<service
    android:name=".DisplayLeakService"
    android:process=":leakcanary"
    android:enabled="false"
    />
<activity
    android:theme="@style/leak_canary_LeakCanary.Base"
    android:name=".internal.DisplayLeakActivity"
    android:process=":leakcanary"
    android:enabled="false"
    android:label="@string/leak_canary_display_activity_label"
    android:icon="@mipmap/leak_canary_icon"
    android:taskAffinity="com.squareup.leakcanary.${applicationId}"
    >
  <intent-filter>
    <action android:name="android.intent.action.MAIN"/>
    <category android:name="android.intent.category.LAUNCHER"/>
  </intent-filter>
</activity>
```

我们可以看到 `DisplayLeakActivity` 被设置为了 Launcher，并设置上了对应的图标，所以我们使用 LeakCanary 会在系统桌面上生成 Icon 入口。但是 `DisplayLeakActivity` 的 `enable` 属性默认是 false，所以在桌面上是不会显示入口的。而在发生内存泄漏的时候，LeakCanary 会主动将 `enable` 属性置为 true。

#### LeakCanary 2 都做了些什么

最近 LeakCanary 升级到了 2.x 版本，这是一次完全的重构，去除了 1.x release 环境下引用的空包 leakcanary-android-no-op。并且 Kotlin 语言覆盖高达 99.8%，也再也不需要在 Application 里面做类似下面的代码。

```java
//com.example.leakcanary.ExampleApplication
@Override
public void onCreate() {
    super.onCreate();
    if (LeakCanary.isInAnalyzerProcess(this)) {
        // This process is dedicated to LeakCanary for heap analysis.
        // You should not init your app in this process.
        return;
    }
    LeakCanary.install(this);
}
```

只需要在依赖里面添加这样的代码就可以了。

```groovy
dependencies {
  debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.0-alpha-2'
}
```

初次看到这样的操作，会觉得非常神奇，仔细阅读源码才回发现它竟然使用了一个骚操作：`ContentProvider`。

在 `leakcanary-leaksentry` 模块的 `AndroidManifest.xml `文件中可以看到：

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest
    xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.squareup.leakcanary.leaksentry"
    >

  <application>
    <provider
        android:name="leakcanary.internal.LeakSentryInstaller"
        android:authorities="${applicationId}.leak-sentry-installer"
        android:exported="false"/>
  </application>
</manifest>

```

再经过查看 `LeakSentryInstaller` 可以看到：

```kotlin
package leakcanary.internal

import android.app.Application
import android.content.ContentProvider
import android.content.ContentValues
import android.database.Cursor
import android.net.Uri
import leakcanary.CanaryLog

/**
 * Content providers are loaded before the application class is created. [LeakSentryInstaller] is
 * used to install [leaksentry.LeakSentry] on application start.
 */
internal class LeakSentryInstaller : ContentProvider() {

  override fun onCreate(): Boolean {
    CanaryLog.logger = DefaultCanaryLog()
    val application = context!!.applicationContext as Application
    InternalLeakSentry.install(application)
    return true
  }

  override fun query(
    uri: Uri,
    strings: Array<String>?,
    s: String?,
    strings1: Array<String>?,
    s1: String?
  ): Cursor? {
    return null
  }

  override fun getType(uri: Uri): String? {
    return null
  }

  override fun insert(
    uri: Uri,
    contentValues: ContentValues?
  ): Uri? {
    return null
  }

  override fun delete(
    uri: Uri,
    s: String?,
    strings: Array<String>?
  ): Int {
    return 0
  }

  override fun update(
    uri: Uri,
    contentValues: ContentValues?,
    s: String?,
    strings: Array<String>?
  ): Int {
    return 0
  }
}
```

确实是真的骚，我们都知道 `ContentProvider` 的 `onCreate()` 的调用时机介于 `Application` 的 `attachBaseContext()` 和 `onCreate()` 之间，LeakCanary 这么做，把 init 的逻辑放到库内部，让调用方完全不需要在 `Application` 里去进行初始化了，十分方便。这样下来既可以避免开发者忘记初始化导致一些错误，也可以让我们庞大的 `Application` 代码更加简洁。

参考：<https://www.jianshu.com/p/49239eac7a76>

