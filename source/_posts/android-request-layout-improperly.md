---
title: AppBarLayout 滑动监听异常：requestLayout() improperly called by android.view.View
date: 2020-04-10 17:47:22
categories:
    - Android
tags:
    - 异常处理
---

最近在项目里发现一个关于AppBarLayout 滑动监听异常，就是监听它的滑动事件时，控制台一直在抛如下异常：

>W/View: requestLayout() improperly called by android.view.View{df595e9 V.ED..... ........ 0,726-1080,726 #7f0902d8 app:id/statusBarSpace} during second layout pass: posting in next frame
>W/View: requestLayout() improperly called by com.google.android.material.appbar.AppBarLayout{e6dac58 V.E...... ......I. 0,0-1080,789 #7f090070 app:id/bar} during layout: running second layout pass

<!--more-->

引发原因暂时不清楚，但是根据查询资料，发现只要滚动变化一致的时候不要做处理就能解决了解决代码如下：

```kotlin

bar.addOnOffsetChangedListener(object : AppBarLayout.OnOffsetChangedListener{
            var lastOffset = -1
            override fun onOffsetChanged(appBarLayout: AppBarLayout?, verticalOffset: Int) {
                if (lastOffset == verticalOffset){
                    return
                }
                lastOffset = verticalOffset
                
                \\ 下面写业务代码即可
            }

        })
```
