---
title: Android RecyclerView 解析 大纲篇
date: 2020-05-25 14:16:16
categories:
    - Android
    - 组件拆解
tags:
    - RecyclerView
---

RecyclerView 是 Android 中使用最多的控件之一，因此利用 RV 写出高质量的代码或者对老项目进行性能优化 可能大大提升app 性能，因此了解RecyclerView的性能就很有必要。

<!--more-->

首先我们用一张 RecyclerView 的思维导图来直观的了解它的架构：

![RecyyclerView 思维导图](http://coderfan.codeagles.com/android-recyclerview-summary.png?e=1590401969&token=o8sDcknnEJ98Lnr_NdIO1uHPhg5-4QGXgAevZD23:LQtWWpAmrOpKsNShBL00AE0Pjlk=)

### 前置知识点

RecyclerView 是 ViewGroup 的，因此Android View的绘制流程和事件分发机制同样适用于它，除此之外还需要了解设计模式相关的知识。以下前置知识点可以自行Google搜索或者查看我相关文章(TODO)

- View 绘制流程
    - onMeasure
    - onLayout
    - onDraw
    - requestLayout & invalidate
- View 事件分发机制
    - onTouchEvent
- 设计模式
    - 适配器模式
    - 桥接模式
    - 观察者模式

### RecyclerView 架构

在了解了前置知识点后，我们来看一下 RecyclerView 的架构，通常我们自定义 ViewGoup 的时候 子View(itemView) 的大小、位置、甚至数据模型都是大致确定的，因此可以直接写到对应实现处。但是想想列表的使用场景，我们发现，要实现一个RecyclerView，它既不知道子View大小会是多少、位置摆放在哪里，也不知道填充的数据源模型是什么样的。

那么问题来了，一个什么规则都不知道的控件要怎么定义呢？RecyclerView 给了一个很好的解答。总结起来可以说就是一句话，只要提供一套行之有效的规则，以后都按照这套规则实现就可以了，怎么说？

1. 不知道子View的大小和摆放？那就约定一个LayoutManager，由它去管理 itemView 的绘制，同时提供了三个已经实现的LayoutManager类，事实上这三种LayoutManager已经足以应付大部分开发需求。
2. 不清楚数据源模型？那就借助适配器模式，提供一个Adapter来将数据源转换成 RecyclerView可使用的 ViewHolder 填充视图
3. 为了优雅和美观，RecyclerView还提供了可以自定义动画和装饰的方法，只要实现 ItemAnimator、ItemDecoration即可

下面我会分具体几篇文章对 RecyclerView 进行拆解：

- [Android Recyclerview 原理解析(一) 布局绘制 & Layoutmanager](https://coderfan.cn/2020/05/26/android-recyclerview-layoutmanager/)
- [RecyclerView 缓存复用 Recycler]
- [RecyclerView 滑动场景分析]
- [RecyclerView 装饰类 itemDecoration]
- [itemAnmiator]



