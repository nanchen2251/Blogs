## 每日一问：谈谈对 MeasureSpec 的理解

作为一名 Android 开发，正常情况下对 View 的绘制机制基本还是耳熟能详的，尤其对于经常需要自定义 View 实现一些特殊效果的同学。

网上也出现了大量的 Blog 讲 View 的 `onMeasure()`、`onLayout()`、`onDraw()` 等，虽然这是一个每个 Android 开发都应该知晓的东西，但这一系列实在是太多了，完全不符合咱们短平快的这个系列初衷。

那么，今天我们就来简单谈谈 `measure()` 过程中非常重要的 `MeasureSpec`。

对于绝大多数人来说，都是知道 `MeasureSpec` 是一个 32 位的 int 类型。并且取了最前面的两位代表 Mode，后 30 位代表大小 Size。

相比也非常清楚 `MeasureSpec` 有 3 种模式，它们分别是 `EXACTLY`、`AT_MOST` 和 `UNSPECIFIED`。
>- 精确模式（MeasureSpec.EXACTLY）：在这种模式下，尺寸的值是多少，那么这个组件的长或宽就是多少，对应 `MATCH_PARENT` 和确定的值。
>- 最大模式（MeasureSpec.AT_MOST）：这个也就是父组件，能够给出的最大的空间，当前组件的长或宽最大只能为这么大，当然也可以比这个小。对应 `WRAP_CONETNT`。
>- 未指定模式（MeasureSpec.UNSPECIFIED）：这个就是说，当前组件，可以随便用空间，不受限制。

通常来说，我们在自定义 View 的时候会经常地接触到 `AT_MOST` 和 `EXACTLY`，我们通常会根据两种模式去定义自己的 View 大小，在 `wrap_content` 的时候使用自己计算或者设置的一个默认值。而更多的时候我们都会认为 `UNSPECIFIED` 这个模式被应用在系统源码中。具体就体现在 `NestedScrollView` 和 `ScrollView` 中。

我们看这样一个 XML 文件：
```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.NestedScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/scrollView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/colorAccent"
        android:text="Hello World"
        android:textColor="#fff">
    </TextView>

</android.support.v4.widget.NestedScrollView>
```
在 `NestedScrollView` 里面写了一个充满屏幕高度的 `TextView`，为了更方便看效果，我们设置了一个背景颜色。但我们从 XML 预览中却会惊讶的发现不一样的情况。
![](https://upload-images.jianshu.io/upload_images/3994917-c47baa704e846909.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们所期望的是填充满屏幕的 `TextView`，但实际效果却和 `TextView` 设置高度为 `wrap_content` 如出一辙。

很明显，这一定是高度测量出现的问题，如果我们的父布局是 `LinearLayout`，很明显没有任何问题。所以问题一定出在了 `NestedScrollView` 的 `onMeasure()` 中。

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    if (this.mFillViewport) {
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        if (heightMode != 0) {
            if (this.getChildCount() > 0) {
                View child = this.getChildAt(0);
                LayoutParams lp = (LayoutParams)child.getLayoutParams();
                int childSize = child.getMeasuredHeight();
                int parentSpace = this.getMeasuredHeight() - this.getPaddingTop() - this.getPaddingBottom() - lp.topMargin - lp.bottomMargin;
                if (childSize < parentSpace) {
                    int childWidthMeasureSpec = getChildMeasureSpec(widthMeasureSpec, this.getPaddingLeft() + this.getPaddingRight() + lp.leftMargin + lp.rightMargin, lp.width);
                    int childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(parentSpace, 1073741824);
                    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
                }
            }

        }
    }
}
```
由于我们并没有在外面设置 `mFillViewport` 这个属性，所以并不会进入到 if 条件中，我们来看看 `NestedScrollView` 的 super `FrameLayout` 的 `onMeasure()` 做了什么。
```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    int count = getChildCount();

    final boolean measureMatchParentChildren =
            MeasureSpec.getMode(widthMeasureSpec) != MeasureSpec.EXACTLY ||
            MeasureSpec.getMode(heightMeasureSpec) != MeasureSpec.EXACTLY;
    mMatchParentChildren.clear();

    int maxHeight = 0;
    int maxWidth = 0;
    int childState = 0;

    for (int i = 0; i < count; i++) {
        final View child = getChildAt(i);
        if (mMeasureAllChildren || child.getVisibility() != GONE) {
            measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, 0);
            final LayoutParams lp = (LayoutParams) child.getLayoutParams();
            maxWidth = Math.max(maxWidth,
                    child.getMeasuredWidth() + lp.leftMargin + lp.rightMargin);
            maxHeight = Math.max(maxHeight,
                    child.getMeasuredHeight() + lp.topMargin + lp.bottomMargin);
            childState = combineMeasuredStates(childState, child.getMeasuredState());
            if (measureMatchParentChildren) {
                if (lp.width == LayoutParams.MATCH_PARENT ||
                        lp.height == LayoutParams.MATCH_PARENT) {
                    mMatchParentChildren.add(child);
                }
            }
        }
    }

    // ignore something...
}
```
注意其中的关键方法 `measureChildWithMargins()`，这个方法在 `NestedScrollView` 中得到了完全重写。
```java
protected void measureChildWithMargins(View child, int parentWidthMeasureSpec, int widthUsed, int parentHeightMeasureSpec, int heightUsed) {
    MarginLayoutParams lp = (MarginLayoutParams)child.getLayoutParams();
    int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec, this.getPaddingLeft() + this.getPaddingRight() + lp.leftMargin + lp.rightMargin + widthUsed, lp.width);
    int childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(lp.topMargin + lp.bottomMargin, 0);
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}	
```
我们看到其中有句非常关键的代码：
```java
int childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(lp.topMargin + lp.bottomMargin, 0);
```
`NestedScrollView` 直接无视了用户设置的 MODE，直接采用了 `UNSPECIFIED` 做处理。**经过测试发现，当我们重写 `NestedScrollView` 的这句代码，并且把 MODE 设置为 `EXACTLY` 的时候，我们得到了我们想要的效果，我已经查看 Google 的源码提交日志，并没有找到原因。**

我起初猜想是只有 `UNSPECIFIED` 才能实现滚动效果，但很遗憾并不是这样的。所以在这里抛出这个问题，希望有知情人士能一起讨论。