---
title: z-index层叠顺序小小的分析
date: 2016-04-03 18:21:50
tags:
- css
---

下文如果没有特别说明，`static`元素指的就是设置`postion:static`的元素或者没有设置`position`的元素。设置了`position`的元素指的是设置了除`static`以外值的元素。

<!-- more -->

## 默认规则
如果不对节点设置`position`属性，文档流后面的元素会遮挡住前面的元素。同时对`static`元素设置`z-index`没有任何效果。对已经设置了`position`定位规则，如果没有设置`z-index`则后面的元素仍然会遮住前面的元素。
设置`postion:static`和没有设置是一个效果，元素设置`position`为`static`以外的任何值的元素，会遮住`static`元素。如果不想让其遮挡住`static`元素，可以将`z-index`设置为`-1`

## 从父规则
对于设置了`position`的元素来说，A节点的`z-index`比B节点大或者A节点和B节点一样大且但A在B的后面，那么A的子节点一定会覆盖在B的前面。因为它的父节点就在B前面所以子节点跟随父元素的安排
非常值得注意的是，从父规则仅在显示设置了`z-index`的节点上有效，并且`position`不为`static`

## 参与规则
`z-index`值越大，则显示越靠前。将`z-index`设置为`0`与不设置`z-index`属性没有区别。
两个元素`z-index`设置为同一个值，后面的元素会遮挡住前面的规则。在这种情况下，即使前面的元素的子元素`z-index`较大，它也会被后面的元素遮挡住。

```html
<div id="a" style="position:relative;z-index:0;">
	<div id="a-1" style="position:relative;z-index:2;">A-1</div>
</div>
 
<div id="b" style="position:relative;z-index:0;">
	<div id="b-1" style="position:relative;z-index:1;">B-1</div>
</div>
```
上述代码效果如下
![](/images/1461304407837_2.png)

## 层级树规则
有了前面例子的铺垫，我们在这里引入一个层级树概念，当元素设置了`position`属性，并且`z-index`给了具体赋值，那么这个元素会被放入一个层级树中，通过比较`z-index`来决定显示顺序的先后，我们来看一个代码
```html
<div id="a" style="position:relative;z-index:2;">
	<div id="a-1" style="position:relative;z-index:0;">A-1</div>
</div>
 
<div id="b">
	<div id="b-1" style="position:relative;z-index:1;">B-1</div>
</div>
```
上述代码效果如下
![](/images/1461304409009_3.png)
层级树图表示如下
![](/images/1461304409059_4.png)
图中虽然 A-1`(z-index:0)`的值比 B-1`(z-index:1)`小, 但因为在层级树里 A`(z-index:2)` 和 B-1 在一个层级, 而 A 的值比 B-1 大, 根据从父规则, A-1 显示在 B-1 前面.

## 参与规则2

如果父元素设置了`position`而没有设置`z-index`，那么他们的子元素会按照各自`z-index`来进行排列，父元素的排列先后顺序不再对子元素产生影响。但是在IE6和IE7中，如果没有设置`z-index`的值，那么默认会设置`z-index`为0。所以在IE6和IE7中，还是会先按父元素的顺序排列。
与这种情况类似的是，如果父元素没有设置`position`属性，那么他们的子元素也会按照各自的`z-index`来排列。

##浮动元素

浮动元素会遮住`static`元素，但是会被设置了`position`的元素遮住。并且对浮动元素设置`z-index`没有任何效果。如果不想后面设置了`position`的元素遮挡住浮动元素，可以将后面的元素设置为`z-index:-1`

参考文章：
[CSS z-index 属性的使用方法和层级树的概念](http://www.neoease.com/css-z-index-property-and-layering-tree/)