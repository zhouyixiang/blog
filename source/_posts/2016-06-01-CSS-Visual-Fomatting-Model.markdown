---
layout: post
title:  "CSS Visual Formatting Model"
date:   2016-06-01 21:53:49 +0800
categories: 开发
tags: [css]
---

[TOC]

# CSS Visual Formatting Model

## Containing Block
这个block也是个box，box是其内嵌box的containing block。
> In general, generated boxes act as containing blocks for descendant boxes; we say that a box "establishes" the containing block for its descendants.
> Each box is given a position with respect to its containing block, but it is not confined by this containing block; it may overflow.

## Viewport
页面渲染到canvas上面，用户通过浏览器提供的viewport和页面交互。 Viewport可能比canvas小，浏览器因此需要提供滚动机制。

## Elment, Box
HTML Element首先会映射成为各种类型的box，页面排版是对box的排版。
* 一个Element可能生成多个box，
* 有的box类似段落的一段，
* 有的box类似段落的一行，
* 有的box类似行内的一个字符

## Block-level Box, Block-level element, Block container box, Block box

### Block-level Box
从该Box在Container中的显示形式来看。
Block-level Box参与BFC，也就是说这些Box是按照垂直方向依次摆放的。

### Block-level element
Block-level element (BLE)可能产生多个box，但每个BLE会生成一个principal block-level box。这个主box会包含BLE生成的其它box，这些额外的box会相对于主box进行布局。主box代表整个BLE参与到any positioning scheme。这几个display属性会让元素变成BLE: 'block', 'list-item', 'table'.

### Block container box, Block box
从该Box包含的下级Box来看。

Block container box的定义有点绕。把标准里面的描述顺序重排一下，把描述分成两类：一类针对该box内部的子box，另一类针对该box对外界。
> A block container box either contains only block-level boxes or establishes an inline formatting context and thus contains only inline-level boxes.

上面说明Block container box要么只包含Block-level boxes，要么只包含inline-level boxes，后面这种情况，还专门说建立了一个inline formatting context。那么前一种情况为什么不提会不会建立BFC呢？

> Not all block container boxes are block-level boxes: non-replaced inline blocks and non-replaced table cells are block containers but not block-level boxes.

> Except for table boxes and replaced elements, a block-level box is also a block container box.

上面两段说明inline-block和table cells虽然不是bock-level boxes，但也是block container box。而block-level box也不一定都是block container box。总而言之，block container box主要是看它内部能容纳什么。

> Block-level boxes that are also block containers are called block boxes.

block boxes的定义。

## Block Formatting Context
BFC下面的（一级）box都是block-level box (可能是根据子block-level element生成的，或隐式生成的)。
> In a block formatting context, boxes are laid out one after the other, vertically, beginning at the top of a containing block. The vertical distance between two sibling boxes is determined by the 'margin' properties. Vertical margins between adjacent block-level boxes in a block formatting context collapse.

在一个BFC里面，box的左边界和BFC的左边界接触 （或右边）。
> In a block formatting context, each box's left outer edge touches the left edge of the containing block (for right-to-left formatting, right edges touch). This is true even in the presence of floats (although a box's line boxes may shrink due to the floats), unless the box establishes a new block formatting context (in which case the box itself may become narrower due to the floats).

当有Float元素存在时，可以把block-level box进一步看成一堆的line box，它们总是尽量往左靠齐到BFC边界，除非遇到Float。

例子：对于下面的HTML结构，
```html
<div>
  <img src="">
  <div class="b1">
    ...
  </div>
  <div class="b2">
    <div class="c1">
      ...
    </div>
    <div class="c2">
      ...
    </div>
  </div>
</div>
```
``` css
.b1 {
  border: 10px red solid;
}

.b2 {
  border: 10px green solid;
  //width: 300px;
  overflow: hidden;
}

.c1 {
  border: 10px blue solid;
}

.c2 {
  border: 10px black solid;
}
img {
  float: left;
  //opacity: 0;
}
```

当.b2不使用overflow:hidden建立新的BFC时，布局如下：

!{bfc}[./img/bfc.png ""]

当.b2使用overflow:hidden建立新的BFC时，布局如下：

!{bfc}[./img/new_bfc.png ""]

另外可以注释掉overflow，设置.b2的width小于图片的宽度，这时.b2的内容看上去像是换行开始布局，因为它的宽度覆盖不了图片，因此图片覆盖的部分时空白的。

!{bfc}[./img/small_width.png ""]

可以通过下面的JsFindle去测试这些情况：￼
https://jsfiddle.net/hostapd/bhuqqtq9/

## 如何建立BFC？
> Floats, absolutely positioned elements, block containers (such as inline-blocks, table-cells, and table-captions) that are not block boxes, and block boxes with 'overflow' other than 'visible' (except when that value has been propagated to the viewport) establish new block formatting contexts for their contents.

* float元素
* 绝对定位元素
* block containers && !block level box
* overflow != visible
