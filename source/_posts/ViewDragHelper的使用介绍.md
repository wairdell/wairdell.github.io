---
title: ViewDragHelper的使用介绍
date: 2020-05-19 16:05:39
tags:
---
### ViewDargHelper 简介
ViewDargHelper 是 Google 官方提供的工具类，使用它我们能轻松的在自己的自定 view 里实现拖拽相关的效果。

### ViewDragHelper 的使用
ViewDragHelper 实例需要使用它的静态方法 `create` 方法来创建。一般我们会用下面第一方法来创建，需要传入两个参数:一个是支持拖拽的 ViewGroup，另一个就是发生拖拽相关操作的回调处理对象 `ViewDragHelper.Callback`。还有一个多了一个参数 `sensitivity` 的重载方法，这个多出来的参数 `sensitivity` 代表拖拽的灵敏度，标准为1.0，数值越大，越灵敏。
``` java
public static ViewDragHelper create(@NonNull ViewGroup forParent, @NonNull Callback cb) {
        return new ViewDragHelper(forParent.getContext(), forParent, cb);
    }

public static ViewDragHelper create(@NonNull ViewGroup forParent, float sensitivity,
            @NonNull Callback cb) {
        final ViewDragHelper helper = create(forParent, cb);
        helper.mTouchSlop = (int) (helper.mTouchSlop * (1 / sensitivity));
        return helper;
    }
```

然后我们看看 `ViewDragHelper.Callback` 最常用的以下三个方法，
- tryCaptureView
  tryCaptureView 是唯一 `ViewDragHelper.Callback` 中唯一一个抽象方法，他的返回代表是否捕捉某个 view 的拖拽事件。返回 true 代表捕获，返回 false 则不捕获。只有捕获的 view 才能触发下面两个方法。
- clampViewPositionHorizontal 和 clampViewPositionVertical
  
    这两个方法都代表着用户对某个 view 有拖拽的动作，而返回值则决定了 view 的位置(通过返回值我们可以限制 view 拖拽的位置)。clampViewPositionHorizontal 代表着水平方法的拖拽，clampViewPositionVertical 代表着竖直方向的拖拽，它们都有三个参数：第一个 View 就是拖动的 View；第二个参数就是用户拖动到某个位置的值；第三个参数 dx 可以理解为滑动的速度，单位是 px 每秒。

```kotlin
var callback = object : ViewDragHelper.Callback() {

            override fun tryCaptureView(child: View, pointerId: Int): Boolean {
                Log.d(TAG, "tryCaptureView() called with: child = [$child], pointerId = [$pointerId]")
                return true
            }

            override fun clampViewPositionHorizontal(child: View, left: Int, dx: Int): Int {
                Log.d(TAG, "clampViewPositionHorizontal() called with: child = [$child], left = [$left], dx = [$dx]")
                //return super.clampViewPositionHorizontal(child, left, dx)
                return left
            }

            override fun clampViewPositionVertical(child: View, top: Int, dy: Int): Int {
                Log.d(TAG, "clampViewPositionVertical() called with: child = [$child], top = [$top], dy = [$dy]")
                return top
                //return super.clampViewPositionVertical(child, top, dy)
            }

```

然后我们还需要在自己自定义的 ViewGroup 中把 touch 相关的事件交给 ViewDragHelper 处理。以下代码是一个 ViewGroup 里面所有的 child 都会根据用户的手指移动。我们用这个小例子来完整了解 ViewDragHelper 的简单使用
```kotlin
class SimpleDragHelperView(context: Context, attrs: AttributeSet?) : FrameLayout(context, attrs) {

    companion object {
        var TAG = "SimpleDragHelperView"
    }

    var viewDragHelper : ViewDragHelper? = null

    constructor(context: Context) : this(context, null)

    init {
        var callback = object : ViewDragHelper.Callback() {

            override fun tryCaptureView(child: View, pointerId: Int): Boolean {
                Log.d(TAG, "tryCaptureView() called with: child = [$child], pointerId = [$pointerId]")
                return true
            }

            override fun clampViewPositionHorizontal(child: View, left: Int, dx: Int): Int {
                Log.d(TAG, "clampViewPositionHorizontal() called with: child = [$child], left = [$left], dx = [$dx]")
                //return super.clampViewPositionHorizontal(child, left, dx)
                return left
            }

            override fun clampViewPositionVertical(child: View, top: Int, dy: Int): Int {
                Log.d(TAG, "clampViewPositionVertical() called with: child = [$child], top = [$top], dy = [$dy]")
                return top
                //return super.clampViewPositionVertical(child, top, dy)
            }

            override fun onViewReleased(releasedChild: View, xvel: Float, yvel: Float) {
                Log.d(TAG, "onViewReleased() called with: releasedChild = [$releasedChild], xvel = [$xvel], yvel = [$yvel]")
                super.onViewReleased(releasedChild, xvel, yvel)
            }

            override fun onEdgeDragStarted(edgeFlags: Int, pointerId: Int) {
                Log.d(TAG, "onEdgeDragStarted() called with: edgeFlags = [$edgeFlags], pointerId = [$pointerId]")
                super.onEdgeDragStarted(edgeFlags, pointerId)
            }
        }
        viewDragHelper = ViewDragHelper.create(this, callback)
        viewDragHelper?.setEdgeTrackingEnabled(ViewDragHelper.EDGE_BOTTOM)
    }

    override fun onInterceptTouchEvent(ev: MotionEvent): Boolean {
        return viewDragHelper?.shouldInterceptTouchEvent(ev) ?: super.onInterceptTouchEvent(ev)
    }

    override fun onTouchEvent(event: MotionEvent): Boolean {
        viewDragHelper?.processTouchEvent(event)
        return true
    }

}
```
### ViewDargHelper 中还有以下几个方法比较常用
#### DragHelperView 的方法
- setEdgeTrackingEnabled(int edgeFlags) 设置这个你就可以在 Callback 监听到 ViewGroup 边缘的滑动了。前面讲的都是手势选中了某个 ViewGroup 中子 view 产生的拖拽，但如果子 view 一开始不在 ViewGroup 可视范围内呢? 如 QQ 的侧滑菜单，这个时候就应该设置这个开关。
- captureChildView(@NonNull View childView, int activePointerId) 主动捕获 view 的拖拽
- settleCapturedViewAt(int finalLeft, int finalTop) 将捕获的视图放置在给定的位置（左，上）。需要我们重写 ViewGroup 的 computeScroll 方法主动调用 continueSettling 方法
  
#### DragHelperView.Callback 中的回调方法
- onViewReleased() 这个是 View 释放拖拽的时候会回调
- onEdgeDragStarted() ViewGroup 边缘的拖拽开始

让我们用下来这个小例子来了解下上面这些方法的具体使用
```kotlin
class TopDragHelperLayout(context: Context, attrs: AttributeSet?) : FrameLayout(context, attrs) {

    companion object {
        var TAG = "TopDragHelperLayout"
    }

    var viewDragHelper: ViewDragHelper? = null

    var contentView: TextView? = null
    var topView: TextView? = null

    init {
        contentView = TextView(context)
        addView(
            contentView,
            LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT)
        )
        contentView?.setBackgroundColor(Color.parseColor("#00FF00"))

        topView = TextView(context)
        addView(
            topView,
            LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                300
            )
        )
        topView?.setBackgroundColor(Color.parseColor("#FF0000"))

        var helperCallback = object : ViewDragHelper.Callback() {

            var currentTop: Int = 0

            override fun tryCaptureView(child: View, pointerId: Int): Boolean {
                Log.d(TAG, "tryCaptureView() called with: child = [$child], pointerId = [$pointerId]")
                return child == topView
            }

            override fun clampViewPositionVertical(child: View, top: Int, dy: Int): Int {
                Log.d(TAG, "clampViewPositionVertical() called with: child = [$child], top = [$top], dy = [$dy]")
                currentTop = Math.min(0, top)
                return currentTop
            }

            override fun onEdgeDragStarted(edgeFlags: Int, pointerId: Int) {
                Log.d(TAG, "onEdgeDragStarted() called with: edgeFlags = [$edgeFlags], pointerId = [$pointerId]")
                viewDragHelper?.captureChildView(topView!!, pointerId)
            }

            override fun onViewReleased(releasedChild: View, xvel: Float, yvel: Float) {
                Log.d(
                    TAG,
                    "onViewReleased() called with: releasedChild = [$releasedChild], xvel = [$xvel], yvel = [$yvel]"
                )
                super.onViewReleased(releasedChild, xvel, yvel)
                topView?.also {
                    when {
                        yvel > 300 -> viewDragHelper?.settleCapturedViewAt(0, 0)
                        yvel < -300 -> viewDragHelper?.settleCapturedViewAt(0, -it.measuredHeight)
                        currentTop > -it.measuredHeight / 2 -> viewDragHelper?.settleCapturedViewAt(0, 0)
                        else -> viewDragHelper?.settleCapturedViewAt(0, -it.measuredHeight)
                    }
                    invalidate()
                }
            }
        }
        viewDragHelper = ViewDragHelper.create(this, helperCallback)
        viewDragHelper?.setEdgeTrackingEnabled(ViewDragHelper.EDGE_TOP)
    }

    override fun onInterceptTouchEvent(ev: MotionEvent): Boolean {
        return viewDragHelper?.shouldInterceptTouchEvent(ev) ?: super.onInterceptTouchEvent(ev)
    }

    override fun onTouchEvent(event: MotionEvent): Boolean {
        viewDragHelper?.processTouchEvent(event)
        return true
    }

    override fun computeScroll() {
        super.computeScroll()
        if (viewDragHelper?.continueSettling(true) == true) {
            invalidate()
        }
    }

    override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
        contentView?.run {
            layout(0, 0, measuredWidth, measuredHeight)
        }
        topView?.layout(0, (topView?.measuredHeight ?: 0) * -1, topView?.measuredWidth ?: 0, 0)
    }


}
```
