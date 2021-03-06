title: 数据结构－图
comments: true
toc: true
date: 2016-07-13 14:47:56
tags: [图]
categories: [数据结构]
---

<!-- more -->
> 这里的`图`是数据结构的图，印象非常浅，所以挖挖坟，从感性认识开始。

先从熟悉的想， `树`是图的一种，是一种无向图。从这里就可以想到图的区分，有向或者是无向，接着就会想到能不能形成回路的特性。尽量多的来看图，少看树。


## 概念

* `图` 是由定点的 `有穷非空集合`     和定点之间边的集合组成。通常的表示 G(V,E) G表示图，V是顶点集合，E是边的集合
    + 图中的点叫做 `顶点`，图中不能没有顶点
    + 定点之间的关系由 `边` 来描述
* `无向图` 任意两个顶点之间的边都是 没有方向之分的，该图成为 无向图。
* `有向图` 任意两个顶点之间的边都是有方向的，例如 `A->B`，该图成为 有向图。有向边也叫做 `弧`
* `无向完全图` 在无向图中，如果任意两个顶点之间都存在边，称为无向完全图。 n个顶点求边 `n(n-1)/2`
* `有向完全图` 在有向图中，如果任意两个顶点之间都存在方向互为相反的两个弧，该图称为有向完全图。 n个顶点求边 `n(n-1)`
* `稀疏图` 和 `稠密图` 又很少边或者弧的图叫做稀疏图，反之称为稠密图，没有量化定义，相对而言。
* `权` 有些图的边或弧具有与它相关的数字，这种和边，弧相关的数叫做权，权可以表示两个顶点之间的距离或者耗费
* `网` 带权的图称为网


## 学习资料
* [数据结构之图](https://www.zybuluo.com/guoxs/note/249812#数据结构之图) 写的非常详尽，没看完，感觉不错。
