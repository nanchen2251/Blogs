## 每日一问：详细说一下 MeasureSpec.UNSPECIFIED

[前面的文章](https://www.jianshu.com/p/6cdbb418df46) 我留下了一个疑惑，那就是到底为什么 `NestedScrollView` 要把子 View 的测量模式强行设置为 `MeasureSpec.UNSPECIFIED` ，这不，在鸿洋的 "wanAndroid" 中，他再次提出了这样的问题：

>MesureSpec.UNSPECIFIED
>1. 这个模式什么时候会遇到？
>2. 遇到后怎么处理？
>3. 有什么注意事项？

下面摘自用户「陈小缘啦啦啦」的回答，我觉得回答的非常到位，特别在这里和大家分享一下。

**`UNSPECIFID`，就是未指定的意思，在这个模式下父控件不会干涉子 View 想要多大的尺寸。**
那么，这个模式什么时候会`onMeasure()` 里遇到呢？其实是取决于它的父容器。
就拿最常用的 `RecyclerView` 做例子，在 `Item` 进行 `measure()` 时，如果列表可滚动，并且 `Item` 的宽或高设置了 `wrap_content` 的话，那么接下来，itemView 的 `onMeasure( )`方法的测量模式就会变成 `MeasureSpec.UNSPECIFIED`。
我们不妨打开 `RecyclerView` 源码，会在 `getChildMeasureSpec()` 方法里看到这么一句注释：
> MATCH_PARENT can't be applied since we can scroll in this dimension, wrap instead using UNSPECIFIED.

它想表达的是：在可滚动的`ViewGroup`中，不应该限制 Item 的尺寸（如果是水平滚动，就不限制宽度），为什么呢？ 因为是可以滚动的，就算 Item 有多宽，有多高，通过滚动也一样能看到滚动前被遮挡的部分。

> 这里其实也就回答了我之前询问的 `NestedScrollView` 要强行设置 Item 为 UNSPECIFIED 的原因。
有同学可能会有疑问： 我设置 `wrap_content`，在 `onMeasure()` 中应该收到的是 `AT_MOST` 才对啊，为什么要强制变成 `UNSPECIFIED`？

**这是因为考虑到 Item 的尺寸有可能超出这个可滚动的 `ViewGroup` 的尺寸，而在 `AT_MOST` 模式下，你的尺寸不能超出你所在的 `ViewGroup` 的尺寸，最多只能等于，所以用 `UNSPECIFIED`会更合适，这个模式下你想要多大就多大。**

那么，我们在自定义 View 的时候，在测量时发现是 `UNSPECIFIED` 模式时，应该怎么做呢？

这个就比较自由了，既然尺寸由自己决定，那么我可以写死为 50，也可以固定为 200。但还是建议结合实际需求来定义咯。

比如 `ImageView`，它的做法就是：有设置图片内容(drawable)的话，会直接使用这个 drawable 的尺寸，但不会超过指定的 `MaxWidth` 或 `MaxHeight`， 没有内容的话就是 0。而 `TextView` 处理 `UNSPECIFIED` 的方式，和 `AT_MOST` 是一样的。

当然了，这些尺寸都不一定等于最后 `layout` 出来的尺寸，因为最后决定子 `View` 位置和大小的，是在 `onLayout()` 方法中，在这里你完全可以无视这些尺寸，去 `layout()`成自己想要的样子。不过，一般不会这么做。