## 每日一问：简述 View 的绘制流程

Android 开发中经常需要用一些自定义 View 去满足产品和设计的脑洞，所以 View 的绘制流程至关重要。网上目前有非常多这方面的资料，但最好的方式还是直接跟着源码进行解读，每日一问系列一直追求短平快，所以本文笔者尽量精简。

想必大多数 Android 开发都知道自定义 View 需要关注的几个方法：`onMeasure()`、`onLayout()` 和 `onDraw()`，这其实也是每个 View 至关重要的绘制流程。

基本绘制都是会从根视图 `ViewRoot` 的 `performTraversals()` 方法开始，从上到下遍历整个视图树，每个View控件负责绘制自己，而 ViewGroup 还需要负责通知自己的子 View 进行绘制操作。`performTraversals()` 的核心代码如下：
```java
private void performTraversals() {
    ...
    int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width);
    int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height);
    ...
    //执行测量流程
    performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
    ...
    //执行布局流程
    performLayout(lp, desiredWindowWidth, desiredWindowHeight);
    ...
    //执行绘制流程
    performDraw();
}
```

#### measure()
```java
public final void measure(int widthMeasureSpec, int heightMeasureSpec)
```
每个 View 都有自己的大小，所以基本自定义 View 的时候都需要重写 `onMeasure()` 这个方法，以定制化我们的 View 的宽高。**如果不重写这个方法，我们通常会出现 `wrap_content` 和 `match_parent` 是一样的显示效果。**至于原因，其实一探源码便知。
```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}

public static int getDefaultSize(int size, int measureSpec) {
    int result = size;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);

    switch (specMode) {
    case MeasureSpec.UNSPECIFIED:
        result = size;
        break;
    case MeasureSpec.AT_MOST:
    case MeasureSpec.EXACTLY:
        result = specSize;
        break;
    }
    return result;
}

protected int getSuggestedMinimumHeight() {
    return (mBackground == null) ? mMinHeight : max(mMinHeight, mBackground.getMinimumHeight());
}		
```
可以看到，`View` 默认是会使用 `getDefaultSize()` 方法进行设置宽高的，在  `AT_MOST` 和 `EXACTLY` 两种情况下都会直接使用测量规格里面的尺寸。在 `UNSPECIFIED` 模式下会直接取`getSuggestedMinimumWidth()` 的返回值。
>`getSuggestedMinimumWidth()`  会直接根据是否设置 `backgroud` 来进行计算，需要注意的是，直接设置 color 作为 `backgroud` 也会直接采用 `minXXX` 的值。

在 `ViewGroup` 中，并没有去重写 `View` 的 `onMeasure()` 方法，而这都需要它的子类根据自己的逻辑去实现，比如 `LinearLayout` 和 `RelativeLayout` 明显测量逻辑是不一样的。不过，`ViewGroup` 倒是提供了一个 `measureChildren()` 方法来依次遍历每个子 View 对其进行测量。

在经过 `onMeasure()` 操作后，`getMeasureWidth()` 和 `getMeasureHeight()` 方法就可以拿到正确的返回值了。

>由于 View 的 measure 过程和 Activity 的生命周期方法不是同步执行的，如果 View 还没有测量完毕，那么获得的宽/高就是 0。所以在 `onCreate()`、`onStart()`、`onResume()` 中均无法正确得到某个 View 的宽高信息。可以通过在 `onWindowFocusChanged()` 判断获取到焦点后进行获取，或者使用 `view.post()` 方式。
#### layout()
```java
public void layout(int l, int t, int r, int b)
```
我们可以重写的 `onLayout()` 方法主要作用是确定子 View 的显示位置，由于 View 已经是最小的层级，所以我们在自定义 View 的时候通常不需要管这个方法，而在自定义 ViewGroup 的时候就不得不注意这个方法了。

经过 `onLayout()` 流程后，我们的 `left`、`right`、`top`、`bottom` 得以赋值，所以这时候可以通过 `getWidth()` 和 `getHeight()` 方法来获取 View 的实际宽高了。
>注意：在 View 的默认实现中，View 的测量宽/高和最终宽/高是相等的，只不过测量宽/高形成于 View 的 `measure` 过程，而最终宽/高形成于 View 的 `layout` 过程，即两者的赋值时机不同，测量宽/高的赋值时机稍微早一些。在一些特殊的情况下则两者不相等：

#### draw()
```java
public void draw(Canvas canvas)
```
绘制的流程也就是通过调用 View 的 `draw()` 方法实现的。`draw()` 方法里的逻辑看起来更清晰，我就不贴源码了。一般是遵循下面几个步骤：
- 绘制背景 – `drawBackground()`
- 绘制自己 – `onDraw()`
- 绘制孩子 – `dispatchDraw()`
- 绘制装饰 – `onDrawScrollbars()`

由于不同的控件都有自己不同的绘制实现，所以V iew 的 `onDraw()` 方法肯定是空方法。而 ViewGroup 由于需要照顾子 View 的绘制，所以肯定在 `dispatchDraw()` 方法里遍历调用了child的 `draw()` 方法。

参考：
[Android View的绘制流程](https://jsonchao.github.io/2018/10/28/Android%20View%E7%9A%84%E7%BB%98%E5%88%B6%E6%B5%81%E7%A8%8B/)
[https://blog.csdn.net/yisizhu/article/details/51527557](https://blog.csdn.net/yisizhu/article/details/51527557)