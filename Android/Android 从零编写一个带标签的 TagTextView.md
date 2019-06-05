最近公司的项目升级到了 9.x，随之而来的就是一大波的更新，其中有个比较明显的改变就是很多板块都出了一个带标签的设计图，如下：
![](https://upload-images.jianshu.io/upload_images/3994917-5e4b79ae3fc5da00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/3994917-28c02d01fb147887.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 怎么实现
看到这个，大多数小伙伴都能想到这就是一个简单的图文混排，不由得会想到鸿洋大佬的图文并排控件 [MixtureTextView](https://github.com/hongyangAndroid/MixtureTextView)，或者自己写一个也不麻烦，只需要利用 shape 背景文件结合 `SpannableString` 即可。

确实如此，利用  `SpannableString` 确实是最方便快捷的方式，但稍不注意这里可能会踩坑。

```kotlin
private fun convertViewToBitmap(view: View): Bitmap {
    view.isDrawingCacheEnabled = true
    view.measure(View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED), View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED))
    view.layout(0, 0, view.measuredWidth, view.measuredHeight)
    view.buildDrawingCache()
    val bitmap = view.drawingCache
    view.isDrawingCacheEnabled = false
    view.destroyDrawingCache()
    return bitmap
}

fun setTagText(style: Int, content: String) {
    val view = LayoutInflater.from(context).inflate(R.layout.layout_codoon_tag_textview, null)
    val tagView = view.findViewById<CommonShapeButton>(R.id.tvName)
    val tag = when (style) {
        STYLE_NONE -> {
            ""
        }
        STYLE_CODOON -> {
            tagView.setStrokeColor(R.color.tag_color_codoon.toColorRes())
            tagView.setTextColor(R.color.tag_color_codoon.toColorRes())
            "自营"
        }
        STYLE_JD -> {
            tagView.setStrokeColor(R.color.tag_color_jd.toColorRes())
            tagView.setTextColor(R.color.tag_color_jd.toColorRes())
            "京东"
        }
        STYLE_TM -> {
            tagView.setStrokeColor(R.color.tag_color_tm.toColorRes())
            tagView.setTextColor(R.color.tag_color_tm.toColorRes())
            "天猫"
        }
        STYLE_PDD -> {
            tagView.setStrokeColor(R.color.tag_color_pdd.toColorRes())
            tagView.setTextColor(R.color.tag_color_pdd.toColorRes())
            "拼多多"
        }
        STYLE_TB -> {
            tagView.setStrokeColor(R.color.tag_color_tb.toColorRes())
            tagView.setTextColor(R.color.tag_color_tb.toColorRes())
            "淘宝"
        }
        else -> {
            ""
        }
    }
    val spannableString = SpannableString("$tag$content")
    val bitmap = convertViewToBitmap(view)
    val drawable = BitmapDrawable(resources, bitmap)
    drawable.setBounds(0, 0, tagView.width, tagView.height)
    spannableString.setSpan(CenterImageSpan(drawable), 0, tag.length, Spannable.SPAN_INCLUSIVE_INCLUSIVE)
    text = spannableString
    gravity = Gravity.CENTER_VERTICAL
}

companion object {
    const val STYLE_NONE = 0
    const val STYLE_JD = 1
    const val STYLE_TB = 2
    const val STYLE_CODOON = 3
    const val STYLE_PDD = 4
    const val STYLE_TM = 5
}
```

xml 文件的样式就不必在这里贴了，很简单，就是一个带 shape 背景的 TextView，不过由于 shape 文件的极难维护性，在我们的项目中统一采用的是自定义 View 来实现这些圆角等效果。
>详细参考作者 blog：[Android 项目中 shape 标签的整理和思考](https://xiaozhuanlan.com/topic/3205781694)

圆角 shape 等效果不是我们在这里主要讨论的东西，我们来看这个代码，思路也是很清晰简洁：首先利用 `LayoutInflater` 返回一个 `View`，然后对这个 `View` 经过一系列判断逻辑确认里面的显示文案和描边颜色等处理。然后通过 `View` 的 `buildDrawingCache()` 的方法生成一个 Bitmap 供 `SpannableString` 使用，然后再把 `spannableString` 设置给 `textView` 即可。

### 一些注意点
其中有个细节需要注意的是，利用 `LayoutInflater` 生成的 `View` 并没有经过 `measure()` 和 `layout()` 方法的洗礼，所以一定没对它的 `width` 和 `height` 等属性赋值。

所以我们在 `buildDrawingCache()` 前做了至关重要的两步操作：
```kotlin
view.measure(View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED), View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED))
view.layout(0, 0, view.measuredWidth, view.measuredHeight)
```

从 `buildDrawingCache()` 源码中我们可以看到，这个方法并不是一定会返回到正确的 `Bitmap`，在我们的 `View` 的 `CacheSize` 大小超过了某写设备的默认值的时候，可能会返回 null。
> 系统给我了我们的默认最大的 `DrawingCacheSize` 为屏幕宽高乘积的 4 倍。

由于我们这里的 View 是极小的，所以暂时没有出现返回 null 的情况。

尽管上面的代码经过测试，基本上能在大部分机型上满足需求。但本着被标记 `@Deprecated` 的过时方法，我们坚决不用的思想，我们需要对生成 `Bitmap` 的方法进行小范围改造。

在最新的 SDK 中，我们发现 `View` 的 `buildDrawingCache()` 等一系列方法都已经被标记了 `@Deprecated` 。
```java
/**
 * <p>Calling this method is equivalent to calling <code>buildDrawingCache(false)</code>.</p>
 *
 * @see #buildDrawingCache(boolean)
 *
 * @deprecated The view drawing cache was largely made obsolete with the introduction of
 * hardware-accelerated rendering in API 11. With hardware-acceleration, intermediate cache
 * layers are largely unnecessary and can easily result in a net loss in performance due to the
 * cost of creating and updating the layer. In the rare cases where caching layers are useful,
 * such as for alpha animations, {@link #setLayerType(int, Paint)} handles this with hardware
 * rendering. For software-rendered snapshots of a small part of the View hierarchy or
 * individual Views it is recommended to create a {@link Canvas} from either a {@link Bitmap} or
 * {@link android.graphics.Picture} and call {@link #draw(Canvas)} on the View. However these
 * software-rendered usages are discouraged and have compatibility issues with hardware-only
 * rendering features such as {@link android.graphics.Bitmap.Config#HARDWARE Config.HARDWARE}
 * bitmaps, real-time shadows, and outline clipping. For screenshots of the UI for feedback
 * reports or unit testing the {@link PixelCopy} API is recommended.
 */
@Deprecated	
public void buildDrawingCache() {
    buildDrawingCache(false);
}		
```
从官方注释中我们发现，**使用视图渲染已经过时，硬件加速后中间缓存很多程度上都是不必要的，而且很容易导致性能的净损失。**

所以我们采用 `Canvas` 进行简单改造一下：
```kotlin
private fun convertViewToBitmap(view: View): Bitmap? {
    view.measure(View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED), View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED))
    view.layout(0, 0, view.measuredWidth, view.measuredHeight)
    val bitmap = Bitmap.createBitmap(view.measuredWidth, view.measuredHeight, Bitmap.Config.ARGB_4444)
    val canvas = Canvas(bitmap)
    canvas.drawColor(Color.WHITE)
    view.draw(canvas)
    return bitmap
}
```
### 突如其来的崩溃
perfect，但很不幸，在上 4.x 某手机上测试的时候，发生了一个空指针崩溃。
 ![](https://upload-images.jianshu.io/upload_images/3994917-cdf2996bff53f732.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
一看日志，发现我们在执行 `view.measure(View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED), View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED))` 这句代码的时候抛出了系统层源码的 bug。

进入源码发现在 `RelativeLayout` 的 `onMeasure()` 中有这样一段代码。
```java
if (isWrapContentWidth) {
    // Width already has left padding in it since it was calculated by looking at
    // the right of each child view
    width += mPaddingRight;

    if (mLayoutParams != null && mLayoutParams.width >= 0) {
        width = Math.max(width, mLayoutParams.width);
    }

    width = Math.max(width, getSuggestedMinimumWidth());
    width = resolveSize(width, widthMeasureSpec);
    // ...
    }
}
```
看起来没有任何问题，但对比 4.3 的源码，发现了一点端倪。
```java
if (mLayoutParams.width >= 0) {
      width = Math.max(width, mLayoutParams.width);
}
```
原来空指针报的是这个 `layoutParams`。
再看看我们 `inflate()` 的代码。
```kotlin
val view = LayoutInflater.from(context).inflate(R.layout.layout_codoon_tag_textview, null)
```
对任何一位 Android 开发来讲，都是最熟悉的代码了，意思很简单，从 xml 中实例化 `View` 视图，但是父视图为 null，所以从 xml 文件实例化的 `View` 视图没办法 `attach` 到 `View` 层次树中，所以导致了 `layoutParams` 这个参数为 null。
既然找到了原因，那么解决方案也就非常简单了。
只需要在 `inflate()` 后，再设置一下 `params` 就可以了。
```kotlin
view.layoutParams = ViewGroup.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT)
```

至此，基本已经实现，主要逻辑代码为：
```kotlin
/**
 * 电商专用的 TagTextView
 * 后面可以拓展直接设置颜色和样式的其他风格
 *
 * Author: nanchen
 * Email: liusl@codoon.com
 * Date: 2019/5/7 10:43
 */
class CodoonTagTextView @JvmOverloads constructor(
        context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0
) : AppCompatTextView(context, attrs, defStyleAttr) {

    private var tagTvSize: Float = 0f

    init {
        val array = context.obtainStyledAttributes(attrs, R.styleable.CodoonTagTextView)
        val style = array.getInt(R.styleable.CodoonTagTextView_codoon_tag_style, 0)
        val content = array.getString(R.styleable.CodoonTagTextView_codoon_tag_content)
        tagTvSize = array.getDimension(R.styleable.CodoonTagTextView_codoon_tag_tv_size, 0f)
        content?.apply {
            setTagText(style, this)
        }
        array.recycle()
    }

    private fun convertViewToBitmap(view: View): Bitmap? {
//        view.isDrawingCacheEnabled = true
        view.measure(View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED), View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED))
        view.layout(0, 0, view.measuredWidth, view.measuredHeight)
//        view.buildDrawingCache()
//        val bitmap = view.drawingCache
//        view.isDrawingCacheEnabled = false
//        view.destroyDrawingCache()
        val bitmap = Bitmap.createBitmap(view.measuredWidth, view.measuredHeight, Bitmap.Config.ARGB_4444)
        val canvas = Canvas(bitmap)
        canvas.drawColor(Color.WHITE)
        view.draw(canvas)
        return bitmap
    }

    fun setTagText(style: Int, content: String) {
        val view = LayoutInflater.from(context).inflate(R.layout.layout_codoon_tag_textview, null)
        view.layoutParams = ViewGroup.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT)
        val tagView = view.findViewById<CommonShapeButton>(R.id.tvName)
        val tag = when (style) {
            STYLE_NONE -> {
                ""
            }
            STYLE_CODOON -> {
                tagView.setStrokeColor(R.color.tag_color_codoon.toColorRes())
                tagView.setTextColor(R.color.tag_color_codoon.toColorRes())
                "自营"
            }
            STYLE_JD -> {
                tagView.setStrokeColor(R.color.tag_color_jd.toColorRes())
                tagView.setTextColor(R.color.tag_color_jd.toColorRes())
                "京东"
            }
            STYLE_TM -> {
                tagView.setStrokeColor(R.color.tag_color_tm.toColorRes())
                tagView.setTextColor(R.color.tag_color_tm.toColorRes())
                "天猫"
            }
            STYLE_PDD -> {
                tagView.setStrokeColor(R.color.tag_color_pdd.toColorRes())
                tagView.setTextColor(R.color.tag_color_pdd.toColorRes())
                "拼多多"
            }
            STYLE_TB -> {
                tagView.setStrokeColor(R.color.tag_color_tb.toColorRes())
                tagView.setTextColor(R.color.tag_color_tb.toColorRes())
                "淘宝"
            }
            else -> {
                ""
            }
        }
        if (tag.isNotEmpty()) {
            tagView.text = tag
            if (tagTvSize != 0f) {
                tagView.textSize = tagTvSize.toDpF()
            }
//            if (tagHeight != 0f) {
//                val params = tagView.layoutParams
//                params.height = tagHeight.toInt()
//                tagView.layoutParams = params
//            }
        }
        val spannableString = SpannableString("$tag$content")
        val bitmap = convertViewToBitmap(view)
        bitmap?.apply {
            val drawable = BitmapDrawable(resources, bitmap)
            drawable.setBounds(0, 0, tagView.width, tagView.height)
            spannableString.setSpan(CenterImageSpan(drawable), 0, tag.length, Spannable.SPAN_INCLUSIVE_INCLUSIVE)
        }
        text = spannableString
        gravity = Gravity.CENTER_VERTICAL
    }

    companion object {
        const val STYLE_NONE = 0    // 不加
        const val STYLE_JD = 1      // 京东
        const val STYLE_TB = 2      // 淘宝
        const val STYLE_CODOON = 3  // 自营
        const val STYLE_PDD = 4     // 拼多多
        const val STYLE_TM = 5      // 天猫
    }
}

```