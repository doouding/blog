---
title: 理解line-height和vertical-align
date: 2016-04-21 19:56:45
tags:
---

我想吐嘈的是，W3C的标准不是一般的晦涩，特别是中文翻译，看了好几遍仍然搞不懂在说什么。只好硬着头皮去看英文。果然我还是太年轻了，看英文看得我递归懵逼。不过再看了几篇前辈的文章之后再回来看，感觉好像理解了一点了。我争取用比较易懂的语言描述一下`line-height`和`vertical-align`属性。

阅读这篇文章之前需要对IFC、行内盒等相关知识点有一个概念。不然会一脸懵逼。

## 行盒的高度计算
**行盒（line box）**说白了就是用来包含视觉上一行文字的一个盒子。为什么说视觉上呢，因为如果一串文字太长了在浏览器里一行显示不下然后换行了，那么这就相当于有两个行盒存在了，因为这个时候有两行文字存在了。行盒是由**行内格式化上下文（IFC）**创建的，但一般我们不需要关注这些，因为浏览器会处理这些。我们需要关心的只是**行盒（line box）**的高度。

行盒的高度计算步骤如下：
1. 计算行盒所包含的的所有**行内级盒（inline-level box）**对于**行内盒（inline box）**，它的高度就是line-height属性的高度。对于替换元素，或者`display`属性为`inline-block`和`inline-table`的元素，它的高度就是盒边界的高度（就是垂直方向上`margin` + `padding` + `content`的宽度总和）
2. 行内级元素根据其`vertical-align`属性对齐。
3. 如果`vertical-align`的值为`top`或`bottom`，那么行盒的高度在计算的时候应该尽可能小。
3. 行盒最终的高度取决于行盒内最高行内级盒的高度。

## 行距与半行距
`line-height`的高度包含字体的高度和行距的高度。我们用`AD`表示一个文字的高度<sup>[1]</sup>，用`L`表示行距。在CSS标准里面，有这样一个公式 `L = 'line-height' - AD`。也就是行距等于`line-height`的高度减去字体的高度。这样就能够得到行距的大小。行距一分为二，一半加在`A`上面也就是基线上方，一半加在`D`上面也就是基线下方。因为行距被平均一分为二，所以在视觉表现上面看起来就和居中一样。所以`inline-height`也可以用于垂直居中。L也可以为负数，这样的效果就是上下同时减去相等的高度。

<span style="color: grey;font-size:14px">[1]：在字体系统中，A表示文字的高度，D表示文字的深度，通过A和D相加就能得到字体本身的高度，A和D交界处就是基线（baseline）的位置。我们不需要纠结A和D的概念，只需要知道A+D是文字的高度，以及文字的高度在基线处一分为二就是了<span>

## 属性分析
我们的前置概念介绍完了，所以现在开始真正的介绍`line-height`和`vertical-align`两个属性了。

### line-height属性
`line-height`属性有四种取值：
- `normal`：默认值，这个默认值实际上是由浏览器设置的一个 "合理" 的`<number>`类型值，因此和`<number>`的效果一样，W3C建议这个值为1.0到1.2之间
- `<length>`：直接指定`line-height`为一个绝对值，要加`px`单位，负值无效
- `<number>`：`font-size`的值乘上这个值来作为`line-height`的高度值，没有单位，负值无效
- `<percentage>`: `font-size`乘上这个值来作为`line-height`的高度值，单位%，负值无效

然后这个属性有一个关键的性质：继承。不同类型的值，继承的方式不同。
对于`<length>`类型，子元素回直接继承父元素的值。

对于`<number>`类型的值，子元素会直接继承父元素的值，当子元素和父元素的`font-size`不一样时，最终计算出来的行高也不一样。
```html
<div class="father" style="line-height: 1.5; font-size: 18px">
  <span class="child" style="font-size: 20px"></span>
  <!-- 父元素的行高计算后是 18 * 1.5 = 27px -->
  <!-- 子元素的行高计算后是 20 * 1.5 = 30px -->
</div>
```

对于`<percentage>`类型的值，子元素会直接继承父元素计算后的值
```html
<div id="father" style="line-height:200%;font-size:15px">
    <span id="child" style="font-size:18px">哈哈</span>
    <!--父元素的行高实际值是 15 * 200% = 30px -->
    <!--子元素直接继承父元素计算后的值也就是30px -->
</div>
```

### vertical-align 属性
`vertical-align`用来设定行内级盒在行盒中的垂直位置。

我们前面说过，行高是由行盒里面包含的最高的那个盒决定的。在CSS规范里面，这个最高的盒被称为"支撑（strut）"。我们不妨称它为支撑盒。

`vertical-align`有一个重要的特性，就是它有些取值最后计算出来的位置，都是相对于这个支撑盒而确定的。

以下取值是相对于支撑盒确定的：

 - `baseline`：默认值，将盒的基线与父盒的基线对齐。若盒没有基线，将底边界与父盒的基线对齐。
 - `middle`，将盒的垂直中心点与父盒的基线加半个父盒中字母「x」的高度<sup>[2]</sup>对齐
 - `sub`，将盒的基线下降到父盒下标的位置
 - `super`，将盒的基线上升到父盒上标的位置
 - `text-top`，将盒的顶边与父盒内容区域的顶边对齐
 - `text-bottom`，将盒的底边与父盒内容区域的底边对齐
 - `<percentage>`，以`line-height`作为参考，`baseline`作为初始位置，上升或下降计算后的距离
 - `<length>`：baseline作为初始位置，上升或下降设定的距离

 下面这两个值是相对于行盒对齐的
 - `top`：把元素顶端与行盒中最高元素的顶端对齐
 - `bottom`：把元素顶端与行盒中最低元素的顶端对齐

 <span style="color: grey;font-size:14px">2：原文写的是x-height，x-height是英文字体设计中的一个术语词，它的标准高是一个小写字母x的高度单位，详情可以看下[维基百科：X字高](https://zh.wikipedia.org/wiki/X%E5%AD%97%E9%AB%98)</span>