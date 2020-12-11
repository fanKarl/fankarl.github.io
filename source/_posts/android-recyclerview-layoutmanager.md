---
title: Android Recyclerview 原理解析(一) 布局绘制 & Layoutmanager
date: 2020-05-26 11:56:14
categories:
    - Android
    - 组件拆解
tags:
    - RecyclerView
---

LayoutManager 是RecyclerView中的布局管理类，用于管理 itemView的布局，官方默认提供了三种实现：线性布局 LinearLayoutManager、网格布局 GridLayoutManager和 流式布局 StaggeredGridLayoutManager，本篇主要依据 LinearLayout 源码进行解析。

<!--more-->

那么现在应该从哪里开始呢？我们有两个切入点可以选择：

1. 根据View绘制流程开始追踪：OnMeasure和onLayout
2. 通过setAdapter()开始追踪

第一个切入点就不用说了，只要我们在xml里配置了RecyclerView运行代码以后就会触发。我们先来看一下 setAdapter()

### setAdapter(adapter)

下面我直接把 setAdapter 这条链的代码先贴出来

```java
    //step01
    public void setAdapter(@Nullable Adapter adapter) {
        // bail out if layout is frozen
        setLayoutFrozen(false);
        setAdapterInternal(adapter, false, true);
        processDataSetCompletelyChanged(false);
        requestLayout();
    }

```

setAdapter() 还是很简单的，总共调用四个方法，第一个和第三个我们可以忽略，setAdapterInternal 主要是设置adapter及其相关配置，暂时也不用关心，最后一行我们发现它调用了 requestLayout，那就回到切入点 1 的情况了。下面我们来按顺序看一下

### onMeasure

通过requestLayout发起重绘，首先执行onMeasure，代码如下(已经省略不必要代码)

```java
    @Override
    protected void onMeasure(int widthSpec, int heightSpec) {
        if (mLayout == null) {
            defaultOnMeasure(widthSpec, heightSpec);
            return;
        }
        if (mLayout.isAutoMeasureEnabled()) {
            final int widthMode = MeasureSpec.getMode(widthSpec);
            final int heightMode = MeasureSpec.getMode(heightSpec);
            
            mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);

            //...

            if (mState.mLayoutStep == State.STEP_START) {
                dispatchLayoutStep1();
            }

            mLayout.setMeasureSpecs(widthSpec, heightSpec);
            mState.mIsMeasuring = true;
            dispatchLayoutStep2();

            mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);

            if (mLayout.shouldMeasureTwice()) {
                //...
                mState.mIsMeasuring = true;
                dispatchLayoutStep2();
                // now we can get the width and height from the children.
                mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);
            }
        } else {
           //......
        }
    }
```

首先第一行就解答了我们第一个问题：如果不设置 LayoutManager，会在设置完宽高以后直接 return

第二个if判断条件是 mLayout.isAutoMeasureEnabled(), 我们这里以LinearLayoutManager为例，返回的是 true，所以我们只需要关注 if 中的操作，else中直接省略即可

了解以上两点，我们重点看一下几个 dispatchLayoutStepN 函数，别的代码可以忽略了，

##### dispatchLayoutStep1()

该函数主要做一些布局开始前的准备工作
- adapter更新
- 确定执行动画
- 存储当前views的信息
- 有必要的情况下，预先执行一些布局操作，同时存储信息

最后需要把 mState.mLayoutState 状态从START改为LAYOUT。

##### dispatchLayoutStep2()

```java
    /**
     * The second layout step where we do the actual layout of the views for the final state.
     * This step might be run multiple times if necessary (e.g. measure).
     */
    private void dispatchLayoutStep2() {
        startInterceptRequestLayout();
        onEnterLayoutOrScroll();
        mState.assertLayoutStep(State.STEP_LAYOUT | State.STEP_ANIMATIONS);
        mAdapterHelper.consumeUpdatesInOnePass();
        mState.mItemCount = mAdapter.getItemCount();
        mState.mDeletedInvisibleItemCountSincePreviousLayout = 0;

        // Step 2: Run layout
        mState.mInPreLayout = false;
        mLayout.onLayoutChildren(mRecycler, mState);

        mState.mStructureChanged = false;
        mPendingSavedState = null;

        mState.mRunSimpleAnimations = mState.mRunSimpleAnimations && mItemAnimator != null;
        mState.mLayoutStep = State.STEP_ANIMATIONS;
        onExitLayoutOrScroll();
        stopInterceptRequestLayout(false);
    }
```

这一步是真正在进行布局的关键，可能会执行多次，比如onMeasure中if中最后一个条件语句就是根据 mLayout.shouldMeasureTwice() 判断是否要再执行一次

既然是关心真正布局的地方，那我们根据注释可以看到run layout注释行下调用了 **mLayout.onLayoutChildren(mRecycler, mState)** 

稍后我们再看具体实现，先返回onMeasure，执行完毕setMeasuredDimensionFromChildren，这是从children 拿到的宽高参数，重新设置后，onMeasure 执行完毕。

### onLayout

看完了onMeasure，我们来看一下OnLayout

```java
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        TraceCompat.beginSection(TRACE_ON_LAYOUT_TAG);
        dispatchLayout();
        TraceCompat.endSection();
        mFirstLayoutComplete = true;
    }
```

代码很少，很直观，我们直接关注 dispatchLayout()

```java
    void dispatchLayout() {
        if (mAdapter == null) {
            Log.e(TAG, "No adapter attached; skipping layout");
            // leave the state in START
            return;
        }
        if (mLayout == null) {
            Log.e(TAG, "No layout manager attached; skipping layout");
            // leave the state in START
            return;
        }
        mState.mIsMeasuring = false;
        if (mState.mLayoutStep == State.STEP_START) {
            dispatchLayoutStep1();
            mLayout.setExactMeasureSpecsFrom(this);
            dispatchLayoutStep2();
        } else if (mAdapterHelper.hasUpdates() || mLayout.getWidth() != getWidth()
                || mLayout.getHeight() != getHeight()) {
            // First 2 steps are done in onMeasure but looks like we have to run again due to
            // changed size.
            mLayout.setExactMeasureSpecsFrom(this);
            dispatchLayoutStep2();
        } else {
            // always make sure we sync them (to ensure mode is exact)
            mLayout.setExactMeasureSpecsFrom(this);
        }
        dispatchLayoutStep3();
    }
```
同样这里也需要根据一系列条件判断是否要执行 dispatchLayoutStepN 函数，step1和 step2我们已经看过，直接看最后的 dispatchLayoutStep3()

##### dispatchLayoutStep3()

该函数的作用跟 step1 很类似，不同的是调用时机是在 step2执行之后，也就是真正布局完成后，用于存储最终的动画信息，以及重置一些必要参数

至此onLayout也简单看完了，抛开细节就显得清晰多了。最后还有一个onDraw方法这里就不再看了，它主要跟ItemDecoration 使用有关

但是到目前为止我们好像遇到的都是 RecyclerView 自身的绘制流程，并没有接触到处理 itemView相关的细节，所以这就轮到 前面留的一个坑了，下面我们回到 dispatchLayoutStep2() 看一看是怎么处理 itemView 的布局的

### mLayout.onLayoutChildren(mRecycler, mState)

mLayout.onLayoutChildren(mRecycler, mState) 在 LayoutManager 中是空实现，这说明它将关于itemView具体布局方式交给了 itemView去实现， 这也就解释了为什么我们可以通过重写 LayoutManager来自定义布局方式

依据 LinearLayoutManager 中该方法实现，官方给出了布局算法的解释

```java
        // layout algorithm:
        // 1) by checking children and other variables, find an anchor coordinate and an anchor item position.
        // 2) fill towards start, stacking from bottom
        // 3) fill towards end, stacking from top
        // 4) scroll to fulfill requirements like stack from bottom.create layout state
```
大致意思就是通过检查 children和其他变量，得到一个坐标锚点和锚点 item 的索引，然后从这个锚点开始，向上布局的话就从下往上填充， 从锚点往下布局，就从上往下布局。

```java
    public void onLayoutChildren(RecyclerView.Recycler recycler, RecyclerView.State state) {
    
        //...
        if (mAnchorInfo.mLayoutFromEnd) {
            // fill towards start
            updateLayoutStateToFillStart(mAnchorInfo);
            mLayoutState.mExtraFillSpace = extraForStart;
            fill(recycler, mLayoutState, state, false);
            startOffset = mLayoutState.mOffset;
            final int firstElement = mLayoutState.mCurrentPosition;
            if (mLayoutState.mAvailable > 0) {
                extraForEnd += mLayoutState.mAvailable;
            }
            // fill towards end
            updateLayoutStateToFillEnd(mAnchorInfo);
            mLayoutState.mExtraFillSpace = extraForEnd;
            mLayoutState.mCurrentPosition += mLayoutState.mItemDirection;
            fill(recycler, mLayoutState, state, false);
            endOffset = mLayoutState.mOffset;
            
            //...
        }
        
        //...
    }

```

这段代码很长，我们需要关注的只有一个 fill 函数，这里如果找不到也没关系，可以尝试逆向思维，从Adapter.onCreateViewHolder 那边向前追溯，这里就不再演示了。

#### fill(recycler, mLayoutState, state, false);

```java
int fill(RecyclerView.Recycler recycler, LayoutState layoutState,
            RecyclerView.State state, boolean stopOnFocusable) {
        // max offset we should set is mFastScroll + available
        final int start = layoutState.mAvailable;
        //...
        int remainingSpace = layoutState.mAvailable + layoutState.mExtraFillSpace;
        LayoutChunkResult layoutChunkResult = mLayoutChunkResult;
        while ((layoutState.mInfinite || remainingSpace > 0) && layoutState.hasMore(state)) {
            layoutChunkResult.resetInternal();
            if (RecyclerView.VERBOSE_TRACING) {
                TraceCompat.beginSection("LLM LayoutChunk");
            }
            layoutChunk(recycler, state, layoutState, layoutChunkResult);
            //...
        }
        if (DEBUG) {
            validateChildOrder();
        }
        return start - layoutState.mAvailable;
    }
```

fill 函数开始填充布局，while 循环处可以看出它是在不停的循环填充大块布局直到剩余空间用完为止，但是我们并没有看到直接取出来children进行计算的代码，于是我们考虑是在 layoutChunk 函数中进行的，继续追下去

#### layoutChunk(recycler, state, layoutState, layoutChunkResult)

```java
    void layoutChunk(RecyclerView.Recycler recycler, RecyclerView.State state,
            LayoutState layoutState, LayoutChunkResult result) {
        View view = layoutState.next(recycler);
        if (view == null) {
            //...
            return;
        }
        RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) view.getLayoutParams();
        if (layoutState.mScrapList == null) {
            if (mShouldReverseLayout == (layoutState.mLayoutDirection
                    == LayoutState.LAYOUT_START)) {
                addView(view);
            } else {
                addView(view, 0);
            }
        } else {
            if (mShouldReverseLayout == (layoutState.mLayoutDirection
                    == LayoutState.LAYOUT_START)) {
                addDisappearingView(view);
            } else {
                addDisappearingView(view, 0);
            }
        }
        measureChildWithMargins(view, 0, 0);
        result.mConsumed = mOrientationHelper.getDecoratedMeasurement(view);
        int left, top, right, bottom;
        if (mOrientation == VERTICAL) {
            //...
        } else {
            //...
        }
        layoutDecoratedWithMargins(view, left, top, right, bottom);
        // Consume the available space if the view is not removed OR changed
        if (params.isItemRemoved() || params.isItemChanged()) {
            result.mIgnoreConsumed = true;
        }
        result.mFocusable = view.hasFocusable();
    }
```

果然第一行就出现了 获取View的代码,到这一步为止我们不再深入去追究如何获取到Children，那是Recycler做的事情，不是本篇关注的重点，我们来梳理剩下的代码

1. 拿到View以后addView，也就是Children是在此时被添加到 RecyclerView的。
2. measureChildWithMargins(view, 0, 0) 对child进行测量，要把间距都计算在内的
3. 计算结果要通过helper类存入到LayoutChunkResult,方便上一层循环计算可使用剩余空间
4. 之后根据 mOrientation 来计算绘制坐标点 top、left、right、bottom
5. 计算完毕调用layoutDecoratedWithMargins(view, left, top, right, bottom)对该child进行布局

至此 RecyclerView 的布局管理主线就梳理清楚了。下面是画的简易流程图

![RecyclerView LayoutManager](http://coderfan.codeagles.com/android-recyclerview-layoutmanager.png)













