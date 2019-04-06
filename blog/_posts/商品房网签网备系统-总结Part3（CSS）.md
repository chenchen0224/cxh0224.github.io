---
title: 商品房网签网备系统-总结Part3（CSS）
date: 2017-09-25 21:29:30
categories: 项目总结
tags:
- CSS
- 商品房网签网备系统
---
本篇是商品房网签网备系统-总结的第三部分：CSS，内容比较零散，包括CSS优先级、媒体查询、viewport等。
<!--more-->

# CSS优先级
- 不同级别的
!important > 内联样式表（标签内部）> 嵌入样式表（当前文件中）> 外部样式表（外部文件中）。
- 同一级别的
按照权值进行计算，规则如下：
!important最高 > id = 100 > class = 10 > 标签 = 1 > 继承 = 0.1

做项目时，发现自己在CSS优先级这块有很大的盲点。首先是对!important有点陌生，然后就是权重的计算犯了低级的错误。
栗子：
```
//CSS
.test {
    color: pink;
}
div.test {
    color: blue;
}
//html
<div class="test">div</div>
<p class="test">段落</p>
```
上边的这个栗子，.test的权重是10，div.test的权重是10+1=11，因此div.test的权重是高于.test的。
![](/img/商品房网签网备系统/class.jpg)
如果要使用!important，需要写在分号前面，上边的栗子可以这样子写color: pink !important;


# 媒体查询@media
@media可以针对不同的媒体类型、不同的屏幕尺寸定义不同的样式。特别是如果你需要设置设计响应式的页面，@media是非常有用的。
## 语法
```
@media mediatype and|not|only (media feature) {
    CSS-Code;
}
```
- 以@media开头
- @media后面的是零个或者多个表达式，用and/not/only连接。
    - 必须以媒体类型作为第一个表达式。此处为screen，还可以是print（打印机）
    - 所有的表达式都应包含在括号内，除非只有一个单词
- 当所有的条件表达式结果为真时，执行{}里的样式

>小贴士：{}里的样式相当于对原有样式进行重写，因此必须***提高权重覆盖掉原有样式***。
提高权重的几种方法：!important（尽量少用）、类选择器改用id选择器、权值++等等。

## 应用
本项目在实现页面响应式和打印时都用到了媒体查询。
### 打印页面
打印页面时，通常需要打印页面的一部分而非全部，因此我们需要将不需要打印的元素隐藏起来。
```
@media print {
  //媒体查询，打印
  .noprint {
    display: none;
  }
}
```
这条查询的媒体类型为print（打印机）

*源码请戳☛☛☛
[https://github.com/huaz224/realEstate/blob/master/src/App.vue](https://github.com/huaz224/realEstate/blob/master/src/App.vue)
line 61-72*

### 响应式页面
不管是PC端还是移动端，针对不同的屏幕尺寸，媒体查询都可以检测到。
```
@media screen and (max-height: 400px) {
  //媒体查询，解决绝对定位与手机软键盘冲突
  #footer {
    display: none;
  }
}
```
这条媒体查询的含义是：当屏幕高度小于等于400px时，也就是手机软键盘出现时，id为footer的页脚要隐藏起来。

*源码请戳☛☛☛
[https://github.com/huaz224/realEstate/blob/master/src/App.vue](https://github.com/huaz224/realEstate/blob/master/src/App.vue)
line 74-165*

# viewport
在做移动端适配时，遇到一个问题，PC的页面在手机屏幕上只显示了一部分，剩余的部分需要滚动才可以看到，而且input输入框一点击页面就放大却缩不回去。度了很多资料，原来是不了解移动端的viewport。
## 一些单位和概念
在介绍viewport之前，先来罗列下可能看到过、迷糊过、放弃过的一些单位和概念：
- 物理像素、逻辑像素
    - dp、pt：（device independent pixels），物理像素，显示屏幕的最小物理单位（固定大小，不会改变）
    我们平常所说的分辨率就是物理像素
    - px：（css pixel），逻辑像素，开发中使用的抽象单位（可以根据不同的设备变大变小）

- dpr
设备像素比（devicePixelRatio），等于物理像素与逻辑像素在数量上的比值
```
dpr = 物理像素 / 逻辑像素（某一方向上）
```
    比如：iphone5 的分辨率是 640 \* 1136 （物理像素），开发中应该是320 \* 568 （逻辑像素），原因是dpr = 2，即一个css像素由4个物理像素渲染。
![](/img/商品房网签网备系统/pixel.jpg)

- ppi
屏幕上每英寸的像素点的数量，即屏幕像素密度。ppi越高，像素数越高，越清晰。
注：ppi是用物理像素算，而不是px
比如：iphone5 的分辨率是 640 \* 1136
![](/img/商品房网签网备系统/ppi.png)


## viewport
一个PC端的页面在移动设备上显示的效果是怎样的呢？？？我们来看下面这张图片
![](/img/商品房网签网备系统/viewport.jpg)
两个viewport将手机屏幕分成两层：
- 底层是布局viewport，页面渲染在底层。
 通过document.documentElement.clientWidth获取（IE则是document.body.clientWidth）
- 顶层是可视viewport，通过缩放来控制可视区域
通过window.innerWidth（IE通过document.documentElement.clientWidth）

由上图可以看到，PC端的页面默认填充的是布局viewport，它的宽度可能远远大于手机屏幕。我们在可视viewport中看到的仅仅是整个页面的一部分内容，需要滑动横向滚动条才可以看到全部页面。
那么，我们怎样让布局viewport的宽度 = 可视viewport的宽度即屏幕宽度呢？？？请看下一小节

## meta标签
在HTML的head标签中加入
```
<meta name="viewport" content="width=device-width, initial-scale=1" />
```
该meta标签的作用是：让当前viewport的宽度（默认布局viewport）等于device-width。
其中，width可以取定值，也可以=device-width，一般为了兼容更多设备，让布局viewport=device-width。
device-width又是什么鬼捏？？？
大多数情况下为设备自己的宽度，可以用window.innerWidth来获取，iOS, android基本上默认都是980px。


设置initial-scale=1后，可视viewport=布局viewport

栗子：
```
//css
body {
    margin: 0;
}
.test {
    width: 320px;
    height: 568px;
    background-color: pink;
    color: #fff;
    font-size: 60px;
    text-align: center;
    line-height: 568px;
    font-family: cursive;
}
//html
<div class="test">320*568</div>
```
没有加meta标签时，效果见图
![](/img/商品房网签网备系统/common.png)
添加meta标签后，效果见图
![](/img/商品房网签网备系统/meta.png)　

移动web最佳viewport配置：
布局viewport = 设备宽度 = 可视viewport




<- - - 本文 ღ 结束 - - - >