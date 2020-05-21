---
title: 自定义RecyclerView的LayoutManager
date: 2020-05-21 10:22:09
tags:
---
### 需要重写的方法
- fun onLayoutChildren(recycler: RecyclerView.Recycler, state: RecyclerView.State)
  
    需要填充整个 RecyclerView。需要回收之前所有的 ItemView,然后根据 RecyclerView 的可视范围将 ItemView add 进来


- fun canScrollHorizontally(): Boolean

    这方法的返回值代表是否可以水平滑动

- fun canScrollVertically(): Boolean 
    
    这个方法的返回值代表是否可以垂直滑动

- fun scrollHorizontallyBy(dx: Int, recycler: RecyclerView.Recycler, state: RecyclerView.State?): Int

    水平滑动时调用，需要在这个方法内通过边界判断是否添加新的 ItemView 和回收 ItemView。dx 代表滑动的距离

### 需要调用的方法

- fun detachAndScrapAttachedViews(recycler : Recycler)

    暂时分离并废弃所有当前附加的子视图。

- fun measureChildWithMargins(child: View, widthUsed: int , heightUsed : int )

    测量 View 的大小

- fun getDecoratedMeasuredWidth(child: View) : Int

    得到 View 测量后测量的宽度

- fun getDecoratedMeasuredHeight(child: View) : Int

    得到 View 测量后测量的高度

- fun layoutDecorated(child: View, left,  top: Int, right: Int, bottom: Int)

    在 RecyclerView 中布置给定的子视图，相当于给定子视图的位置

-  getChildAt(index: Int): View

    根据索引得到当前 RecyclerView 中某个 ItemView

- getPosition(view: View): Int
    
    返回某个 ItemView 对应 adapter 的位置

- removeAndRecycleView(view： View, recycler： recycler)
  
    回收和移除某个 ItemView

- addView(child: View)

    将视图添加到 RecyclerView 中。
### Recycler 中方法

- getViewForPosition(position: Int): View

    从缓存中拿到对应索引的 itemView，这个方法内部会先从 scrap 缓存中取 itemView，如果没有则从 recycler 缓存中取，如果还没有则调用 adapter 的 onCreateViewHolder() 去创建 itemView。

#### 下面是自定义实现水平布局的 LayoutManager，通过下面这个 Demo 我们可以大致了解一个自定义 LayoutManager 怎么写
```kotlin
import android.view.ViewGroup
import androidx.recyclerview.widget.RecyclerView

class SimpleHorizontalLayoutManager : RecyclerView.LayoutManager() {

    override fun generateDefaultLayoutParams(): RecyclerView.LayoutParams {
        return RecyclerView.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT)
    }

    override fun onLayoutChildren(recycler: RecyclerView.Recycler, state: RecyclerView.State) {
        super.onLayoutChildren(recycler, state)
        //如果 itemCount 为0时没有任何 itemView 直接返回
        if (itemCount <= 0) {
            return
        }
        if (state.isPreLayout) {
            return
        }
        //将现在Attached到RecyclerView的移除回收
        detachAndScrapAttachedViews(recycler)
        var useWidth = 0
        for (i in 0..itemCount) {
            //获取某个position的itemView
            val childView = recycler.getViewForPosition(i)
            //添加进recyclerView
            addView(childView)
            //测量itemView
            measureChildWithMargins(childView, 0, 0)
            val childViewWidth = getDecoratedMeasuredWidth(childView)
            val childViewHeight = getDecoratedMeasuredHeight(childView)
            //根据itemView测量的宽高布局
            layoutDecorated(childView, useWidth, 0, useWidth + childViewWidth, childViewHeight)
            useWidth += childViewWidth
            //如果itemView布局已超出recycleView的宽度就不在添加
            if (useWidth >= width) {
                break
            }
        }
    }

    override fun scrollHorizontallyBy(dx: Int, recycler: RecyclerView.Recycler, state: RecyclerView.State): Int {
        var resultDx = 0
        //判断是左滑还是右滑,小于0是左滑
        if (dx < 0) {
            //获取当前recyclerView中第一个itemView
            var childView = getChildAt(0) ?: return 0
            var childViewPosition = getPosition(childView)
            //如果第一itemView的position为0时，则没有前一个元素
            if (childViewPosition == 0) {
                resultDx = if (childView.left < 0) Math.max(childView.left, dx) else 0
            } else if (childView.left < 0) {
                //还在recycler的可以范围内不用加载
                resultDx = dx
            } else {
                //不为0，则有前一个元素，获取前一个位置的itemView
                var newChildView = recycler.getViewForPosition(childViewPosition - 1)
                //添加到recyclerView第一个位置
                addView(newChildView, 0)
                measureChildWithMargins(newChildView, 0, 0)
                var newChildViewWidth = getDecoratedMeasuredWidth(newChildView)
                var newChildViewHeight = getDecoratedMeasuredHeight(newChildView)
                //根据新的itemView测量宽高将itemView布局到之前一个itemView的左边
                layoutDecoratedWithMargins(
                    newChildView,
                    childView.left - newChildViewWidth,
                    0,
                    childView.left,
                    newChildViewHeight
                )
                resultDx = dx
            }
        } else {
            var childView = getChildAt(childCount - 1) ?: return 0
            var childViewPosition = getPosition(childView)
            //如果第一itemView的position为最后一个，则没有下一个元素
            if (childViewPosition == itemCount - 1) {
                //限制最后一个滑动位置
                resultDx = if(childView.right >= width) Math.min(childView.right - width, dx) else 0
            } else if (childView.right >= width) {
                //这里是如果可视范围还在RecyclerView内就不用加载下一个
                resultDx = dx
            } else {
                //不为最后一个，则有下一个元素，获取下一个位置的itemView
                var newChildView = recycler.getViewForPosition(childViewPosition + 1)
                //添加到recyclerView最后面
                addView(newChildView)
                measureChildWithMargins(newChildView, 0, 0)
                var newChildViewWidth = getDecoratedMeasuredWidth(newChildView)
                var newChildViewHeight = getDecoratedMeasuredHeight(newChildView)
                //根据新的itemView测量宽高将itemView布局到之前一个itemView的右面
                layoutDecoratedWithMargins(
                    newChildView,
                    childView.right,
                    0,
                    childView.right + newChildViewWidth,
                    newChildViewHeight
                )
                resultDx = dx
            }
        }
        recyclerView(dx, recycler)
        offsetChildrenHorizontal(-resultDx)
        return resultDx
    }

    private fun recyclerView(dx: Int, recycler: RecyclerView.Recycler) {
        if (dx <= 0) {
            for (i in 0..childCount) {
                var childView = getChildAt(i) ?: continue
                if (childView.left > width) {
                    removeAndRecycleView(childView, recycler)
                }
            }
        } else {
            for(i in 0..childCount) {
                var childView = getChildAt(i) ?: continue
                if(childView.right < 0) {
                    removeAndRecycleView(childView, recycler)
                }
            }
        }
    }

    //可以水平滑动
    override fun canScrollHorizontally(): Boolean {
        return true
    }
}
```