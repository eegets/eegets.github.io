---
layout: post
title:  "Android理解自定义View"
categories: 自定义View
---

当Android SDK中提供的系统UI控件无法满足业务需求时，我们就需要考虑自己实现UI控件。

自定义UI控件有2种方式：
> 1. 继承自系统提供的成熟控件（比如LinearLayout、RelativeLayout、ImageView等）
> 2. 直接继承自系统View或ViewGroup, 并且绘制显示内容。

### 继承自成熟控件
相对而言，这种方式相对简单，因为大部分核心工作，比如控件大小测量，控件位置摆放位置等计算，在系统控件中Google已为我们实现了，我们不需要关心这部分的内容，只需要在基础上进行扩展需求即可。因为基本上比较简单，所以我们这种我们暂时不做研究。

### 继承自View或ViewGroup
这种方式相较第一种麻烦，但是更加灵活，也能实现更加复杂的UI界面。一般情况下使用这种实现方式可以解决以下几个问题：
> 1. 如何根据相应的属性将UI元素绘制到界面
> 2. 如何自定义控件大小，也就是测量布局的宽高
> 3. 如果是ViewGroup，该如何安排其内部子View的摆放位置

以上3个问题依次在如下3个方法中可以得到解决：
> 1. onDraw
> 2. onMeasure
> 3. onLayout

因此自定义View的重点工作就是复写并实现这3个方法。
` 注意：并不是每个自定义View都需要实现这3个方法，大多数情况下实现其中1个或2个就可以满足需求`

我们先来依次研究一下如上3个方法
#### onDraw
onDraw方法接收一个Canvas类型的参数，Canvas可以理解为一个画布，在这块画布上可以绘制各种类型的UI元素。

系统提供了一系列Canvas操作方法，如下：
![Ciqc1F66brqANYwaAAFgenmfG7o790.png](https://upload-images.jianshu.io/upload_images/18406403-4d4a57e80ef748be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上图可以看出，Canvas每次绘制都需要传入一个Paint对象，Paint就相当于一个画笔，我们可以通过画笔的各种属性，来实现不同的绘制效果：
![CgqCHl66bsKAC3aYAAEfignRLSI590.png](https://upload-images.jianshu.io/upload_images/18406403-bad5a9ef9ac6e924.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们通过一个测试代码来看看：

我们首先定义PieImageView继承自View， 在onDraw方法中，分别使用Canvas的drawArc和drawCircle方法绘制弧度和圆形。
```kotlin
class PieImageView @JvmOverloads constructor(
    context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    private var progress: Int = 0
    private val MAX_PROGRESS: Int = 100
    private var arcPaint: Paint? = null
    private var circlePaint : Paint? = null
    private var bound: RectF? = RectF()

    fun setProgress(progress: Int) {
        this.progress = progress
        ViewCompat.postInvalidateOnAnimation(this)
    }

    init {
        arcPaint = Paint(Paint.ANTI_ALIAS_FLAG)
        arcPaint?.style = Paint.Style.FILL_AND_STROKE
        arcPaint?.strokeWidth =  dpToPixel(0.1f, context)
        arcPaint?.color = Color.RED

        circlePaint = Paint(Paint.ANTI_ALIAS_FLAG)
        circlePaint?.style = Paint.Style.STROKE
        circlePaint?.strokeWidth = dpToPixel(2f, context)
        circlePaint?.color = Color.argb(120, 0xff, 0xff, 0xff)
    }

    //布局加载完成执行
    override fun onFinishInflate() {
        super.onFinishInflate()
        Log.d("TAG", "onFinishInflate")
    }

    //布局控件大小发生变化时调用，只在初始化执行一次
    override fun onSizeChanged(w: Int, h: Int, oldw: Int, oldh: Int) {
        super.onSizeChanged(w, h, oldw, oldh)
        val min = Math.min(w, h)
        val max = w + h - min
        val r = Math.min(w, h) / 2
        Log.d(
            "TAG",
            "onSizeChanged w = $w, h = $h, oldW = $oldw, oldH = $oldh, min = $min, max = $max, r = $r"
        )
        val left = ((max shr 1) - r).toFloat()
        val top = ((min shr 1) - r).toFloat()
        val right = ((max shr 1) + r).toFloat()
        val bottom = ((min shr 1) + r).toFloat()
        bound?.set(left, top, right, bottom)
        Log.d(
            "TAG",
            "onSizeChanged bound left = $left, top = $top, right = $right, bottom = $bottom"
        )
    }

    override fun onDraw(canvas: Canvas?) {
        super.onDraw(canvas)
        Log.d("TAG", "onDraw")
        if (progress != MAX_PROGRESS && progress != 0) {
            val angle = progress * 360f / MAX_PROGRESS
            canvas?.drawArc(bound!!, 270f, angle, true, arcPaint!!)
            canvas?.drawCircle(
                bound?.centerX()!!,
                bound?.centerY()!!,
                bound?.height()!! / 2,
                circlePaint!!
            )
        }
    }
}
```
在xml中我们使用上述的PieImageView, 设置宽高为200dp，并在Activity中设置PieImageView的进度为45，如下代码
```xml
<?xml version="1.0" encoding="utf-8"?>#### onMeasure
自定义View为什么要进行测量。正常情况下，我们直接在XML不居中定义好View的宽高，然后让自定义View在此宽高的区域显示即可。但是为了更好的兼容不同尺寸的屏幕，Android系统提供了wrap_content和match_parent属性来规范控件的显示规则。分别代表**自适应大小**和**填充父布局的大小**，但是这两个属性并没有指定具体大小，因此我们需要在onMeasure方法中过滤出这两种情况，真正的测量出自定义View应该显示的宽高大小。

<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <com.eegets.measureview.PieImageView
        android:id="@+id/pieImageView"
        android:layout_width="300dp"
        android:layout_height="200dp"
        tools:ignore="MissingConstraints" />

</FrameLayout>
```
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        pieImageView.setProgress(45)
    }
}
```
运行结果如下图：
![QQ图片20201116165717.png](https://upload-images.jianshu.io/upload_images/18406403-5d9cba355305d8eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


输出一下PieImageView Log日志：
```kotlin
 D/TAG: onFinishInflate
 D/TAG: onSizeChanged w = 788, h = 525, oldW = 0, oldH = 0, min = 525, max = 788, r = 262
 D/TAG: onSizeChanged bound left = 132.0, top = 0.0, right = 656.0, bottom = 524.0
 D/TAG: onDraw
```
从log日志中我们能得到几点信息
* 1、onFinishInflate和onSizeChanged只执行了一次
*  2、w = 788 和 h = 525 对应xml中的 android:layout_width="300dp" android:layout_height="200dp"得到的真实的宽高
`注意：w = 788 和 h = 525 随着手机分辨率的不同值也会不同`
* 3、使用kotlin的`shr`相当于java的`<<`移位运算符限定了bound在界面显示的区域
* 4、设置bound边界的left, top, right, bottom[具体值可以看上图的标注]
> 位移运算符`<<`、`>>`、`>>>`
` <<      :     左移运算符，num << 1,相当于num乘以2`
 `>>      :     右移运算符，num >> 1,相当于num除以2`
`>>>    :     无符号右移，忽略符号位，空位都以0补齐`

如上布局，我们在xml中将PieImageView的宽高设置成了固定值"300dp"和"200dp"，我们尝试将布局设置成自适应`wrap_content`，重新运行显示效果如下：
![WeChat Image_20201116172442.png](https://upload-images.jianshu.io/upload_images/18406403-c7b54538b838f071.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

另外我们也看看此时的日志输出：
```kotlin
D/TAG: onFinishInflate
D/TAG: onSizeChanged w = 1080, h = 1584, oldW = 0, oldH = 0, min = 1080, max = 1584, r = 540
D/TAG: onSizeChanged bound left = 252.0, top = 0.0, right = 1332.0, bottom = 1080.0
D/TAG: onDraw
```
很明显，PieImageView并没有正常显示，并且log日志输出的`right = 1332.0,`很明显大于了屏幕的宽度`w = 1080`，这也是PieImageView超出屏幕没有正常显示的原因。根本原因是PieImageView中没有在onMeasure方法中进行重新测量，并重新设置宽高。

当我们不设置onMeasure时，父view其实已经实现了onMeasure方法，我们看一下父类onMeasure做了什么
```java

protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    //父布局传入宽，高约束
    //通过比较最小的尺寸和父布局传入的尺寸，找出合适的尺寸
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
public static int getDefaultSize(int size, int measureSpec) {
    //size 为默认大小
    int result = size;
    //获取父布局传入的测量模式
    int specMode = MeasureSpec.getMode(measureSpec);
    //获取父布局传入的测量尺寸
    int specSize = MeasureSpec.getSize(measureSpec);

    //根据测量模式选择不同的测量尺寸
    switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            //父布局不对子布局施加任何约束，使用默认尺寸
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY: //[重要代码]
            //使用父布局给的尺寸
            result = specSize;
            break;
    }
    //返回子布局确定后的尺寸
    return result;
}
```
如上代码可以看出，当我们设置了`wrap_content`时，父布局的onMeasure给子View返回了父布局给的尺寸，也就是` [重要代码]`处，也就是上述log日志中输出的`w = 1080`，这也就说明了为什么我们布局的显示是错误的。
#### onMeasure
自定义View为什么要进行测量。正常情况下，我们直接在XML不居中定义好View的宽高，然后让自定义View在此宽高的区域显示即可。但是为了更好的兼容不同尺寸的屏幕，Android系统提供了wrap_content和match_parent属性来规范控件的显示规则。分别代表**自适应大小**和**填充父布局的大小**，但是这两个属性并没有指定具体大小，因此我们需要在onMeasure方法中过滤出这两种情况，真正的测量出自定义View应该显示的宽高大小。

我们首先用一个比喻来看看Measure的测量过程，如下图
![WeChat Image_20201117142811.png](https://upload-images.jianshu.io/upload_images/18406403-c050baf355c16dfd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所有工作都是在 onMeasure 方法中完成，方法定义如下：
```kotlin
override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec)
}
```
可以看出，该方法会传入2个参数`widthMeasureSpec`和`heightMeasureSpec`。这两个参数是父视图传递给子View的两个参数，包含了2种信息：`宽、高`以及`测量模式`。
我们获取一下`宽、高`和测量模式，通过Android SDK中的MeasureSpec.java类获取。代码如下：
```kotlin
override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
    //宽度测量模式
    val widthMode = MeasureSpec.getMode(widthMeasureSpec)
    val heightMode = MeasureSpec.getMode(heightMeasureSpec)

    Log.d(
        "TAG",
        "MeasureSpecMode MeasureSpec.AT_MOST = ${MeasureSpec.AT_MOST}, MeasureSpec.EXACTLY = ${MeasureSpec.EXACTLY}, MeasureSpec.UNSPECIFIED = ${MeasureSpec.UNSPECIFIED}"
    )
    Log.d(
        "TAG",
        "widthMode widthMode = $widthMode, heightMode = $heightMode"
    )

    // 判断是wrap_content的测量模式
    if (MeasureSpec.AT_MOST == widthMode || MeasureSpec.AT_MOST == heightMode) {
        val measuredWidth = MeasureSpec.getSize(widthMeasureSpec)
        val measuredHeight = MeasureSpec.getSize(heightMeasureSpec)
        // 将宽高设置为传入宽高的最小值
        val size = if (measuredWidth > measuredHeight) measuredHeight else measuredWidth
        // 调用setMeasuredDimension设置View实际大小
        setMeasuredDimension(size, size)
        Log.d(
            "TAG",
            "onMeasure +++++ measuredWidth = $size, measureHeight = $size"
        )
    } else {
        setMeasuredDimension(getDefaultSize(suggestedMinimumWidth, widthMeasureSpec), getDefaultSize(suggestedMinimumHeight, heightMeasureSpec))
        Log.d(
            "TAG",
            "onMeasure ----- defaultMeasuredWidth = ${getDefaultSize(suggestedMinimumWidth, widthMeasureSpec)}, defaultMeasuredHeight = ${getDefaultSize(suggestedMinimumHeight, heightMeasureSpec)}"
        )
    }
}
```
同时我们输出一下log日志对比看一下：
```java
D/TAG: MeasureSpecMode MeasureSpec.AT_MOST = -2147483648, MeasureSpec.EXACTLY = 1073741824, MeasureSpec.UNSPECIFIED = 0
D/TAG: widthMode widthMode = -2147483648, heightMode = -2147483648
D/TAG: onMeasure +++++ measuredWidth = 1080, measureHeight = 1080
D/TAG: MeasureSpecMode MeasureSpec.AT_MOST = -2147483648, MeasureSpec.EXACTLY = 1073741824, MeasureSpec.UNSPECIFIED = 0
D/TAG: widthMode widthMode = -2147483648, heightMode = -2147483648
D/TAG: onMeasure +++++ measuredWidth = 1080, measureHeight = 1080
D/TAG: onSizeChanged w = 1080, h = 1080, oldW = 0, oldH = 0, min = 1080, max = 1080, r = 540
D/TAG: onSizeChanged bound left = 0.0, top = 0.0, right = 1080.0, bottom = 1080.0
D/TAG: onDraw
```
可以看到，通过onMeasure进行测量，我们最终在`onSizeChanged`中的`left = 0.0, top = 0.0, right = 1080.0, bottom = 1080.0` right变成了1080，也就是屏幕的宽度
#### ViewGroup中的onMeasure
如果我们自定义的控件是一个容器，onMeasure的测量会更复杂一点，因为ViewGroup在测量自身之前，首先需要测量内部子View所占大小，然后才能确定自己的大小。比如以下代码：
![WeChat Image_20201117150004.png](https://upload-images.jianshu.io/upload_images/18406403-db5f809e1675fa95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图可以看出LinearLayout的最终宽度是由其内部最大的子View宽度决定的。

当我们自定义一个ViewGroup时，也需要在onMeasure中综合考虑子View的宽度。比如要实现一个流式布局FlowLayout，效果如下：
![Ciqc1F66b0uANdyTAASLs9Xvo14469.png](https://upload-images.jianshu.io/upload_images/18406403-0a9e575942799069.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在大多数App的搜索界面经常会用到FlowLayout来展示历史搜索记录以及热门搜索项。
FlowLayout的每一行item个数都不一定，当每行的item累计宽度超过可用总宽度时，则需要重启一行摆放Item。因此我么需要在onMeasure方法中主动的分行计算出FlowLayou的最终高度，代码如下所示：
```kotlin
override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec)
    val widthMode = MeasureSpec.getMode(widthMeasureSpec)
    val widthSize = MeasureSpec.getSize(widthMeasureSpec)
    val heightMode = MeasureSpec.getMode(heightMeasureSpec)
    var heightSize = MeasureSpec.getSize(heightMeasureSpec)

    //获取容器中子View的个数
    val childCount = childCount
    //记录每一行View的总宽度
    var totalLineWidth = 0
    //记录每一行最高View的高度
    var perLineMaxHeight = 0
    //记录当前ViewGroup的总高度
    var totalHeight = 0

    Log.d("TAG", "onMeasure childCount = $childCount")

    for (index in 0 until childCount) {
        val childView = getChildAt(index)
        measureChild(childView, widthMeasureSpec, heightMeasureSpec)
        val lp = childView.layoutParams as MarginLayoutParams
        //获得子View的测量宽度
        val childWidth = childView.measuredWidth + lp.leftMargin + lp.rightMargin
        //获得子VIew的测量高度
        val childHeight = childView.measuredHeight + lp.topMargin + lp.bottomMargin
        Log.d("TAG", "onMeasure totalLineWidth=$totalLineWidth, childWidth=$childWidth, totalLineWidth + childWidth = ${totalLineWidth + childWidth}, widthSize=$widthSize")
        if (totalLineWidth + childWidth > widthSize) {
            //统计总高度
            totalHeight += perLineMaxHeight
            //开启新的一行
            totalLineWidth = childWidth
            perLineMaxHeight = childHeight
            Log.d("TAG", "onMeasure true totalLineWidth=$totalLineWidth, perLineMaxHeight=$perLineMaxHeight, childHeight=$childHeight")
        } else {
            //记录每一行的总宽度
            totalLineWidth += childWidth
            //比较每一行最高的View
            perLineMaxHeight = Math.max(perLineMaxHeight, childHeight)
            Log.d("TAG", "onMeasure false totalLineWidth=$totalLineWidth, perLineMaxHeight=$perLineMaxHeight, childHeight=$childHeight")
           
        }
        //当该View已是最后一个View时，将该行最大高度添加到totalHeight中
        if (index == childCount - 1) {
            totalHeight += perLineMaxHeight
        }

        //如果高度的测量模式是EXACTLY，则高度用测量值，否则用计算出来的总高度（这时高度的设置为wrap_content）
        heightSize = if (heightMode == MeasureSpec.EXACTLY) heightSize else totalHeight
        Log.d(
            "TAG",
            "onMeasure childMargin measuredWidth = $childWidth, leftMargin = ${lp.leftMargin}, rightMargin = ${lp.rightMargin}, totalHeight=$totalHeight, heightSize=$heightSize"
        )
        setMeasuredDimension(widthSize, heightSize)
    }
}
```
上述 onMeasure 方法的主要目的有 2 个：

> 1、调用 measureChild 方法递归测量子 View；
> 2、通过叠加每一行的高度，计算出最终 FlowLayout 的最终高度 totalHeight。

#### onLayout
> 根据之前的思维导图，我们知道，老父亲给三个儿子，老大（老大儿子：儿子）、老二、老三分配了具体的良田面积，三个儿子及老大的儿子也都确认了自己的需要的良田面积。**这就是：Measure过程**

> 既然知道了分配给各个儿孙的良田大小，那他们到底分到哪一块呢，是靠边、还是中间、还是其它位置呢？先分给谁呢？
老父亲想按到这个家的时间先后顺序来吧(对应addView 顺序)，老大是自己的长子，先分配给他，于是从最左侧开始，划出3亩田给老大。现在轮到老二了，由于老大已经分配了左侧的3亩，那么给老二的5亩地只能从老大右侧开始划分，最后剩下的就分给老三。**这就是：ViewGroup onLayout 过程。**
老大拿到老父亲给自己指定的良田的边界，将这个边界(左、上、右、下)坐标记录下来。这就是：View Layout过程
接着老大告诉自己的儿子：你爹我也要为自己考虑哈，从你爷爷那继承的5亩田地不能全分给你，我留一些养老。**这就是设置：padding 过程**
如果老二在最开始测量的时候就想：我不想和老大、老三的田离得太近，那么老父亲就会给老大、老三与老二的土地之间留点缝隙。**这就是设置：margin 过程**

上面的FlowLayout的onMeasure只是算出了ViewGroup的最终显示宽高，但是并没有规定某个子View应该在何处显示、间距是多少。要定义ViewGroup内部子View的显示规则，则需要复写并实现onLayout方法。
onLayout声明如下：
```kotlin
override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
      TODO("Not yet implemented")
}
```
它是一个抽象方法，也就是每一个ViewGroup必须要实现如何排列子View，具体的就是循环遍历子View，调用子View.layout(left, top, right, bottom)来设置布局位置。FlowLayout设置布局代码如下：
```kotlin

/**
 * 摆放控件
 * 通过循环并通过‘totalLineWidth + childWidth > width’进行宽度比较将我们的子View存储到lineViews中，也就是一列能装几个子View
 * 同样通过循环将每一行显示的子View的lineViews存储到MAllViews中，mAllViews中存储了n行lineViews列(每列的个数可能不一致)组成的数组
 *
 * 最后通过遍历mAllViews和lineViews得到子View并通过`childView.layout(leftChild, topChild, rightChild, bottomChild)`摆放到合适的位置
 */
override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
    mAllViews.clear()
    mPerLineMaxHeight.clear()

    //存放每一行的子View
    var lineViews = mutableListOf<View>()
    //记录每一行已存放View的总宽度
    var totalLineWidth = 0

    //记录每一行最高View的高度
    var lineMaxHeight = 0

    /*************遍历所有View，将View添加到List<List></List><View>>集合中</View> */
    Log.d("TAG", "onLayout ")
    //获得子View的总个数
    val childCount = childCount
    for (i in 0 until childCount) {
        val childView: View = getChildAt(i)
        val lp = childView.layoutParams as MarginLayoutParams
        val childWidth: Int = childView.measuredWidth + lp.leftMargin + lp.rightMargin
        val childHeight: Int = childView.measuredHeight + lp.topMargin + lp.bottomMargin
        Log.d("TAG", "onLayout width=$width, totalLineWidth=$totalLineWidth, childWidth=$childWidth, totalLineWidth + childWidth=${totalLineWidth + childWidth}")
        if (totalLineWidth + childWidth > width) {
            mAllViews.add(lineViews)
            mPerLineMaxHeight.add(lineMaxHeight)
            //开启新的一行
            totalLineWidth = childWidth
            lineMaxHeight = childHeight
            lineViews = mutableListOf()
            Log.d("TAG", "onLayout true lineViews size=${lineViews.size}, mAllViews size=${mAllViews.size}")
        } else {
            totalLineWidth += childWidth
            Log.d("TAG", "onLayout false lineViews size=${lineViews.size}, mAllViews size=${mAllViews.size}")
        }
        lineViews.add(childView)
        lineMaxHeight = Math.max(lineMaxHeight, childHeight)
    }
    //单独处理最后一行
    mAllViews.add(lineViews)
    mPerLineMaxHeight.add(lineMaxHeight)
    Log.d(
        "TAG",
        "onLayout mAllViews size=${mAllViews.size}, mPerLineMaxHeight size=${mPerLineMaxHeight.size}, lineViews size=${lineViews.size}"
    )

    /************遍历集合中的所有View并显示出来 */
    //表示一个View和父容器左边的距离
    var mLeft = 0
    //表示View和父容器顶部的距离
    var mTop = 0
    for (i in 0 until mAllViews.size) {
        //获得每一行的所有View
        lineViews = mAllViews[i]
        lineMaxHeight = mPerLineMaxHeight[i]
        for (j in lineViews.indices) {
            val childView: View = lineViews[j]
            val lp = childView.layoutParams as MarginLayoutParams
            val leftChild = mLeft + lp.leftMargin
            val topChild = mTop + lp.topMargin
            val rightChild: Int = leftChild + childView.measuredWidth
            val bottomChild: Int = topChild + childView.measuredHeight
            //四个参数分别表示View的左上角和右下角
            childView.layout(leftChild, topChild, rightChild, bottomChild)
            mLeft += lp.leftMargin + childView.measuredWidth + lp.rightMargin
        }
        mLeft = 0
        mTop += lineMaxHeight
    }
}
```
以上onLayout方法中做了两件事情如下：
> 1、通过循环并通过`totalLineWidth + childWidth > width`进行宽度比较将我们的子View存储到lineViews中，也就是一列能装几个子View
同样通过循环将每一行显示的子View的lineViews存储到MAllViews中，mAllViews中存储了n行lineViews列(每列的个数可能不一致)组成的数组
> 2、通过遍历mAllViews和lineViews得到子View并通过`childView.layout(leftChild, topChild, rightChild, bottomChild)`摆放到合适的位置
FlowLayout调用FlowActivity.kt如下代码：
```kotlin
class FlowActivity :Activity() {
//    private val list = mutableListOf("阿迪达斯", "李林", "耐克", "361", "海蓝之迷面霜", "coach", "fendi", "亚历山大短靴", "二手中古", "Ariete阿丽亚特", "ASH", "阿玛尼牛仔")
    private val list = mutableListOf("阿迪达斯", "李林", "耐克", "361", "海蓝之迷面霜", "coach", "fendi")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_flow)
        addView()
    }

    private fun addView() {
        flowLayout.removeAllViews()
        list.forEach {
            val view = LayoutInflater.from(this).inflate(R.layout.item_flow, flowLayout, false) as TextView
            view.text = it
            flowLayout.addView(view)
        }
    }
}
```
activity_flow.xml
```xml
<com.eegets.measureview.FlowLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:id="@+id/flowLayout"
    android:background="#bbbbbb">

</com.eegets.measureview.FlowLayout>
```
item_flow.xml
```xml
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/itemFlow"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:background="#ff00ff"
    android:paddingTop="10dp"
    android:paddingBottom="10dp"
    android:paddingLeft="20dp"
    android:paddingRight="20dp"
    android:layout_marginTop="5dp"
    android:layout_marginBottom="5dp"
    android:layout_marginLeft="10dp"
    android:layout_marginRight="12dp"/>
```
最终界面展示如下图：
![WeChat Image_20201119103121.png](https://upload-images.jianshu.io/upload_images/18406403-179f13f2c7a08151.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至此我们自定义基本上就研究明白了

源码已上传至Github [https://github.com/eegets/MeasureViewTest](https://github.com/eegets/MeasureViewTest)


最后感谢 大神姜新星的Android进阶
