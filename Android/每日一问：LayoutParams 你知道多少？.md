## 每日一问：LayoutParams 你知道多少？

前面的文章中着重讲解了 View 的测量流程。其中我提到了一句非常重要的话：**View 的测量匡高是由父控件的 `MeasureSpec` 和 View 自身的 `LayoutParams 共同决定的。**我们在前面的 [每日一问：谈谈对 MeasureSpec 的理解](https://www.jianshu.com/p/6cdbb418df46) 把 MeasureSpec 的重点进行了讲解，其实另外一个 LayoutParams 同样是非常非常重要。

#### 从概念讲起
`LayoutParams`，顾名思义，就是布局参数。而且大多数人对此都是司空见惯，我们 XML 文件里面的每一个 View 都会接触到 `layout_xxx` 这样的属性，这实际上就是对布局参数的描述。大概大家也就清楚了，`layout_` 这样开头的东西都不属于 View，而是控制具体显示在哪里。

#### LayoutParams 都有哪些初始化方法
通常来说，我们都会把我们的控件放在 XML 文件中，即使我们有时候需要对屏幕做比较「取巧」的适配，会直接通过 `View.getLayoutParams()` 这样的方法获取 `LayoutParams` 的实例，但我们接触的少并不代表它的初始化方法不重要。

> 实际上，用代码写出来的 View 加载效率要比在 XML 中加载快上大约 1 倍。只是在如今手机配置都比较高的情况下，我们常常忽略了这种方式。

我们来看看 `ViewGroup.LayoutParams` 到底有哪些构造方法。
```java
public LayoutParams(Context c, AttributeSet attrs) {
    TypedArray a = c.obtainStyledAttributes(attrs, R.styleable.ViewGroup_Layout);
    setBaseAttributes(a,
            R.styleable.ViewGroup_Layout_layout_width,
            R.styleable.ViewGroup_Layout_layout_height);
    a.recycle();
}

public LayoutParams(int width, int height) {
    this.width = width;
    this.height = height;
}

public LayoutParams(LayoutParams source) {
    this.width = source.width;
    this.height = source.height;
}

LayoutParams() {  }
```

#### MarginLayoutParams
除去最后一个放给 `MarginLayoutParams` 做处理的方法外，我们在 `ViewGroup` 中还有 3 个构造方法。他们分别负责给 XML 处理、直接让用户指定宽高、还有类似集合的 `addAll()` 这样的方式的赋值方法。

实际上，`ViewGroup` 的子类的 `LayoutParams` 类拥有更多的构造方法，感兴趣的自己翻阅源码查看。在这里我想更加强调一下我上面提到的 `MarginLayoutParams`。

`MarginLayoutParams` 继承于 `ViewGroup.LayoutParams`。

```java
public static class MarginLayoutParams extends ViewGroup.LayoutParams {
    @ViewDebug.ExportedProperty(category = "layout")
    public int leftMargin;

    @ViewDebug.ExportedProperty(category = "layout")
    public int topMargin;

    @ViewDebug.ExportedProperty(category = "layout")
    public int rightMargin;

    @ViewDebug.ExportedProperty(category = "layout")
    public int bottomMargin;

    @ViewDebug.ExportedProperty(category = "layout")
    private int startMargin = DEFAULT_MARGIN_RELATIVE;

    @ViewDebug.ExportedProperty(category = "layout")
    private int endMargin = DEFAULT_MARGIN_RELATIVE;

    public MarginLayoutParams(Context c, AttributeSet attrs) {
        super();
        TypedArray a = c.obtainStyledAttributes(attrs, R.styleable.ViewGroup_MarginLayout);
        setBaseAttributes(a,
                R.styleable.ViewGroup_MarginLayout_layout_width,
                R.styleable.ViewGroup_MarginLayout_layout_height);

        int margin = a.getDimensionPixelSize(
                com.android.internal.R.styleable.ViewGroup_MarginLayout_layout_margin, -1);
        if (margin >= 0) {
            leftMargin = margin;
            topMargin = margin;
            rightMargin= margin;
            bottomMargin = margin;
        } else {
            int horizontalMargin = a.getDimensionPixelSize(
                    R.styleable.ViewGroup_MarginLayout_layout_marginHorizontal, -1);
            // ... something
        }
        // ... something
    }
}
```
一看代码，自然就清楚了，为什么我们以前会发现在 XML 布局里， `layout_margin` 属性的值会覆盖 `layout_marginLeft` 与 `layout_marginRight` 等属性的值。

>实际上，事实上，绝大部分容器控件都是直接继承 `ViewGroup.MarginLayoutParams` 而非 `ViewGroup.LayoutParams`。所以我们再自定义 `LayoutParams` 的时候记得继承 `ViewGroup.MarginLayoutParams` 。

#### 在代码里面使用 LayoutParams
前面介绍了 `LayoutParams` 的几种构造方法，我们下面以 `LinearLayout.LayoutParams` 来看看几种简单的使用方式。
```kotlin
val textView1 = TextView(this)
textView1.text = "不指定 LayoutParams"
layout.addView(textView1)

val textView2 = TextView(this)
textView2.text = "手动指定 LayoutParams"
textView2.layoutParams = LinearLayout.LayoutParams(LinearLayout.LayoutParams.WRAP_CONTENT, LinearLayout.LayoutParams.WRAP_CONTENT)
layout.addView(textView2)

val textView3 = TextView(this)
textView3.text = "手动传递 LayoutParams"
textView3.layoutParams = LinearLayout.LayoutParams(ViewGroup.LayoutParams(100, 100))
layout.addView(textView3)
```
我们看看 `addView()` 都做了什么。
```java
public void addView(View child) {
    addView(child, -1);
}

public void addView(View child, int index) {
    if (child == null) {
        throw new IllegalArgumentException("Cannot add a null child view to a ViewGroup");
    }
    LayoutParams params = child.getLayoutParams();
    if (params == null) {
        params = generateDefaultLayoutParams();
        if (params == null) {
            throw new IllegalArgumentException("generateDefaultLayoutParams() cannot return null");
        }
    }
    addView(child, index, params);
}

@Override
protected LayoutParams generateDefaultLayoutParams() {
    if (mOrientation == HORIZONTAL) {
        return new LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);
    } else if (mOrientation == VERTICAL) {
        return new LayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.WRAP_CONTENT);
    }
    return null;
}

public void addView(View child, int index, LayoutParams params) {
    if (DBG) {
        System.out.println(this + " addView");
    }
    if (child == null) {
        throw new IllegalArgumentException("Cannot add a null child view to a ViewGroup");
    }
    requestLayout();
    invalidate(true);
    addViewInner(child, index, params, false);
}

private void addViewInner(View child, int index, LayoutParams params,
        boolean preventRequestLayout) {

   	// ...

    if (!checkLayoutParams(params)) {
        params = generateLayoutParams(params);
    }

  	// ...
}

@Override
protected boolean checkLayoutParams(ViewGroup.LayoutParams p) {
    return p instanceof LinearLayout.LayoutParams;
}
```
看起来 `ViewGroup` 真是煞费苦心，如果我们没有给 View 设置 `LayoutParams`，则系统会帮我们根据 `orientation` 设置默认的 `LayoutParams`。甚至是我们即使在 `addView()` 之前设置了错误的 `LayoutParams` 值，系统也会我们帮我们进行纠正。
>虽然系统已经做的足够完善，帮我们各种矫正错误，但在 `addView()` 之后，我们还强行设置错误的 `LayoutParams`，那还是一定会报 `ClassCastException` 的。

![](https://upload-images.jianshu.io/upload_images/3994917-00024485ee8afac5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`LayoutParams` 很重要，每一名 Android 开发都应该尽力地去掌握，只有弄清楚了系统的编写方式，应对上面类似简书的流式布局才能更好处理。
> 实际上 Google 出的 [FlexboxLayout](https://github.com/google/flexbox-layout) 已经做的相当完美。
> 当然如果使用的 `RecyclerView`，还可以自己写一个 `FlowLayoutManager` 进行处理。

原文较多地参考自：[https://blog.csdn.net/yisizhu/article/details/51582622](https://blog.csdn.net/yisizhu/article/details/51582622)