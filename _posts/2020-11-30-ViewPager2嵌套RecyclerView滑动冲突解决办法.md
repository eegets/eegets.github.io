---
layout: post
title:  "ViewPager2嵌套RecyclerView滑动冲突解决办法"
categories: ViewPager2 RecycleView
---

在嵌套的滚动视图与包含它的 `ViewPager2` 对象方向相同的情况下，`ViewPager2` 本身并不支持该滚动视图。例如，在垂直方向的 `ViewPager2` 对象内，垂直滚动视图无法滚动。

为了支持方向相同的 `ViewPager2` 对象内的滚动视图，如果您希望改为滚动嵌套的元素，则必须对 `ViewPager2` 对象调用 [`requestDisallowInterceptTouchEvent()`](https://developer.android.google.cn/reference/android/view/ViewGroup?hl=zh_cn#requestDisallowInterceptTouchEvent(boolean))。[ViewPager2 嵌套滚动示例](https://github.com/android/views-widgets-samples/blob/master/ViewPager2/app/src/main/res/layout/item_nested_recyclerviews.xml#L43)展示了一种使用通用[自定义封装容器布局](https://github.com/android/views-widgets-samples/blob/master/ViewPager2/app/src/main/java/androidx/viewpager2/integration/testapp/NestedScrollableHost.kt)解决此问题的办法。

由于ViewPager2是个final类，无法重写，而RecyclerViewImpl是ViewPager2的私有类，也无法被继承，所以要解决滑动冲突，只能通过监听子类RecyclerView的分发事件通过`requestDisallowInterceptTouchEvent`来限制父布类的拦截事件

>Child can scroll, disallow all parents to intercept   
  `parent.requestDisallowInterceptTouchEvent(true)`
  
* 方案一：
使用自定义View包裹子类的RecyclerView，实现如下：
```kotlin
/**
 * Layout to wrap a scrollable component inside a ViewPager2. Provided as a solution to the problem
 * where pages of ViewPager2 have nested scrollable elements that scroll in the same direction as
 * ViewPager2. The scrollable element needs to be the immediate and only child of this host layout.
 *
 * This solution has limitations when using multiple levels of nested scrollable elements
 * (e.g. a horizontal RecyclerView in a vertical RecyclerView in a horizontal ViewPager2).
 */
class NestedScrollableHost : FrameLayout {
    constructor(context: Context) : super(context)
    constructor(context: Context, attrs: AttributeSet?) : super(context, attrs)

    private var touchSlop = 0
    private var initialX = 0f
    private var initialY = 0f
    private val parentViewPager: ViewPager2?
        get() {
            var v: View? = parent as? View
            while (v != null && v !is ViewPager2) {
                v = v.parent as? View
            }
            return v as? ViewPager2
        }

    private val child: View? get() = if (childCount > 0) getChildAt(0) else null

    init {
        touchSlop = ViewConfiguration.get(context).scaledTouchSlop
    }

    private fun canChildScroll(orientation: Int, delta: Float): Boolean {
        val direction = -delta.sign.toInt()
        return when (orientation) {
            0 -> child?.canScrollHorizontally(direction) ?: false
            1 -> child?.canScrollVertically(direction) ?: false
            else -> throw IllegalArgumentException()
        }
    }

    override fun onInterceptTouchEvent(e: MotionEvent): Boolean {
        handleInterceptTouchEvent(e)
        return super.onInterceptTouchEvent(e)
    }

    private fun handleInterceptTouchEvent(e: MotionEvent) {
        val orientation = parentViewPager?.orientation ?: return

        // Early return if child can't scroll in same direction as parent
        if (!canChildScroll(orientation, -1f) && !canChildScroll(orientation, 1f)) {
            return
        }

        if (e.action == MotionEvent.ACTION_DOWN) {
            initialX = e.x
            initialY = e.y
            parent.requestDisallowInterceptTouchEvent(true)
        } else if (e.action == MotionEvent.ACTION_MOVE) {
            val dx = e.x - initialX
            val dy = e.y - initialY
            val isVpHorizontal = orientation == ORIENTATION_HORIZONTAL

            // assuming ViewPager2 touch-slop is 2x touch-slop of child
            val scaledDx = dx.absoluteValue * if (isVpHorizontal) .5f else 1f
            val scaledDy = dy.absoluteValue * if (isVpHorizontal) 1f else .5f

            if (scaledDx > touchSlop || scaledDy > touchSlop) {
                if (isVpHorizontal == (scaledDy > scaledDx)) {
                    // Gesture is perpendicular, allow all parents to intercept
                    parent.requestDisallowInterceptTouchEvent(false)
                } else {
                    // Gesture is parallel, query child if movement in that direction is possible
                    if (canChildScroll(orientation, if (isVpHorizontal) dx else dy)) {
                        // Child can scroll, disallow all parents to intercept
                        parent.requestDisallowInterceptTouchEvent(true)
                    } else {
                        // Child cannot scroll, allow all parents to intercept
                        parent.requestDisallowInterceptTouchEvent(false)
                    }
                }
            }
        }
    }
}
```
使用如上自定义View包裹RecyclerView
```xml
    <androidx.viewpager2.integration.testapp.NestedScrollableHost
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp">
        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/first_rv"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="#FFFFFF" />
    </androidx.viewpager2.integration.testapp.NestedScrollableHost>
```


* 方案二
重写RecyclerView的dispatchTouchEvent方法，实现如下：
```java
public class RecyclerViewAtViewPager2 extends RecyclerView {

    public RecyclerViewAtViewPager2(@NonNull Context context) {
        super(context);
    }

    public RecyclerViewAtViewPager2(@NonNull Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
    }

    public RecyclerViewAtViewPager2(@NonNull Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    private int startX, startY;

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                startX = (int) ev.getX();
                startY = (int) ev.getY();
                getParent().requestDisallowInterceptTouchEvent(true);
                break;
            case MotionEvent.ACTION_MOVE:
                int endX = (int) ev.getX();
                int endY = (int) ev.getY();
                int disX = Math.abs(endX - startX);
                int disY = Math.abs(endY - startY);
                LogUtils.debugInfo("DispatchTouchEvent disX="+ disX + "; disY" + disY + "; canScrollHorizontally(startX - endX) = " + canScrollHorizontally(startX - endX) + "; canScrollVerticallyhttps://developer.android.google.cn/training/animation/vp2-migration?hl=zh_cn(startY - endY)" + canScrollVertically(startY - endY));
                if (disX > disY) {
                    //如果是纵向滑动，告知父布局不进行时间拦截，交由子布局消费，　requestDisallowInterceptTouchEvent(true)
                    getParent().requestDisallowInterceptTouchEvent(canScrollHorizontally(startX - endX));
                } else {
                    getParent().requestDisallowInterceptTouchEvent(canScrollVertically(startX - endX));
                }
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                getParent().requestDisallowInterceptTouchEvent(false);
                break;
        }
        return super.dispatchTouchEvent(ev);
    }
}
```
使用此自定义View代替RecyclerView
```xml
<com.xxx.xxx.xxx.RecyclerViewAtViewPager2
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

refrence:

 [https://developer.android.google.cn/training/animation/vp2-migration?hl=zh_cn](https://developer.android.google.cn/training/animation/vp2-migration?hl=zh_cn)
