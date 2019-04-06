---
title: 仿饿了么APP商家页面-总结Part2（CSS）
date: 2017-01-27 22:45:02
categories: 项目总结
tags:
    - CSS
    - 仿饿了么APP商家页面
---
本篇是商品房网签网备系统-总结的第二部分：CSS，内容比较零散，包括弹性盒子、垂直居中、同一行的inline-block元素间有缝隙、盒模型border-box、移动端1px边框、溢出文本显示省略号的效果、扩大点击区域等等。
<!-- more -->

# 弹性盒子
弹性盒子(Flex Box)是CSS3引入的一种新的布局模式，提供了一种更优雅的方式实现响应式布局。尤其是一些特殊的布局，比如垂直居中。
教程请戳☛☛☛[阮一峰：Flex布局教程](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

这里重点讲一下flex属性：
## flex属性
### 语法
flex属性是flex-grow、flex-shrink、flex-basis的简写，默认值为0 1 auto。我们先假定主轴（flex-direction）为水平轴：
- flex-grow：放大的比例
0（默认值）：不管怎么样都不放大；
1：根据剩余空间来放大。

- flex-shrink：缩小的比例
0：不管怎么样都不缩小；
1（默认值）：当空间不足时会缩小。

- flex-basis：占据空间的基准值
可能的取值：
  1）auto（默认值）：如果该子元素在CSS样式中明确定义width值，则取该值，如果没有，则取子元素的content值；
  2）0%：子元素视为0尺寸的，即使在CSS样式中明确定义width值，也形同虚设；
  3）定值（比如100px）:是子元素未分配剩余空间时占据的尺寸。

我们需要注意的是，
- flex-grow和flex-shrink都是在父容器扣除子元素的flex-basis基准值之后，如果还存在剩余空间，根据伸缩放大系数最后怎么分配剩余空间的一个比例；
- 主轴为水平轴，分配的是宽度（width）；主轴为垂直轴，分配的是高度（height）；
- 当flex取值为一个非负数字，则其他两个属性取值为1，flex-basis取0%。

这里有一篇博客对flex属性讲的很好，建议阅读一下，请戳☛☛☛[flex:1详解](http://blog.csdn.net/sunyyyb/article/details/67633693)

### 项目中的应用
在仿站时，就用Flex属性实现了经典的等分布局、一列定宽，其它列自适应宽度布局。
- 等分布局
等分布局需要将弹性子元素的flex设为1，有几个子元素就会在宽度上几等分弹性父容器。本项目就是用flex:1实现的导航栏3列等分布局。
源码请戳☛☛☛
*[https://github.com/huaz224/ele-sell/blob/master/src/App.vue](https://github.com/huaz224/ele-sell/blob/master/src/App.vue)
line 51-56*

- 一列定宽，其它列自适应宽度布局
先看一个栗子：
    //HTML
    ```
    	<div class="cartbody">
    	    <div class="content-left">购物车、价格、描述</div>
    	    <div class="content-right">去结算</div>
    	</div>
    ```
    //CSS
    ```
        .cartbody{
            display:flex;
        }
        .content-left {
            flex:1;
        }
        .content-right {
            flex: 0 0 105px;
        }    
    ```
    以上代码，将父容器设置为弹性容器，左边的弹性子元素自适应宽度，右边的弹性子元素不放大，不缩小，宽度固定为105px。
    源码请戳☛☛☛
    *[https://github.com/huaz224/ele-sell/blob/master/src/components/shopcart/shopcart.vue](https://github.com/huaz224/ele-sell/blob/master/src/components/shopcart/shopcart.vue)
    line 172、175、239*

# 垂直居中
## 设置父元素的height = line-height
适用于：单行的行内元素
缺点：当行内元素的长度大于块的宽度时，就有内容脱离了块，所以要将内容控制在块的范围之内。

>line-height与font-size的计算值之差，在CSS中成为“行间距”。分为两半，分别加到一个文本行内容的顶部和底部。

## display:table-cell和vertical-align:middle
适用于：多行的行内元素
缺点：注意IE6、7并不支持这个样式，兼容性比较差。不推荐使用这种方式。
栗子1：
//HTML
```
<div class='wrapper'>
    <span>垂直居中???</span>
    <em>多行的行内元素</em>
</div>
```
//CSS
```
    .wrapper {
        width: 100px;
        height: 100px;
        background: pink;
        display: table-cell; 
        vertical-align: middle;
    }
```
也可以这酱紫写:
//HTML
```
    <div class='wrapper'>
        <span class="child">垂直居中???<em>多行的行内元素</em></span>
    </div>
```
//CSS
```
    .wrapper {
        width: 100px;
        height: 100px;
        background: pink;
        display: table;
    }

    .child {
        display: table-cell;
        vertical-align: middle;
    }
```

## 通过盒子模型计算margin、padding值
这需要计算父元素的height和line-height，通过精确计算来确定margin或者padding值来实现垂直居中，这种方法用的最多。

## 使用绝对定位和transform
适用于：未知高度的block元素
我们来看一个栗子：
//HTML
```
    <div class='wrapper'>
        <div class="child">垂直居中???<em>多行的行内元素</em></div>
    </div>
```
//CSS
```
    .wrapper {
        width: 300px;
        height: 300px;
        background: pink;
        position: relative;
    }
    .child {
        position: absolute;
        top: 50%;
        transform: translate(0, -50%);
    }
```

## 使用flex布局
我们来看一个栗子：
//HTML
```
    <div class='wrapper'>
        <div class="child">垂直居中???<span>多行的行内元素</span></div>
    </div>
```
//CSS
```
    .wrapper {
        width: 300px;
        height: 300px;
        background: pink; 
        display: flex;
    }
    .child {
        margin:auto;
    }
```
还可以这样子写：
//CSS
```
    .wrapper {
        width: 300px;
        height: 300px;
        background: pink; 
        display: flex;
        align-items: center;
        justify-content: center;
    }
```


# 同一行的inline-block元素间有缝隙
这个问题是将一个div转换为inline-block时发现的，左边的图片与右边的内容之间就出现了缝隙，去度娘溜了一圈，发现只要是在一行上inline-block元素都有这个问题，缝隙出现的原因在于标签之间的的空格。去除缝隙的方法有两种：
- 手动去掉HTML中的空格
将它们之间的空格直接删掉，这种方法简单暴力，但是降低了代码的可读性，可用性不强。可以这样子写：
```
<span class="text">haha</span
><span class="text">bili</span>
```
	也可以借助注释：
	```
		<span class="text">haha</span><!-- 
		--><span class="text">bili</span>
	```

- font-size: 0;
设置inline-block元素的父容器font-size为0，然后再各自设置它们的font-size就可以啦。
在仿站时，本方法用的最多，这种方法就是不要忘记设置子元素的font-size。

- 通过浮动和绝对定位实现与inline-block一样的布局
这种方法需要对浮动和绝对定位非常熟悉，思路也非常清晰，缺点是，由于浮动和绝对定位会导致元素脱离文档流，这对性能会有一定影响，不适合大面积使用。

本项目也用了1次浮动和绝对定位，源码可戳☛☛☛
*[https://github.com/huaz224/ele-sell/blob/master/src/components/shopcart/shopcart.vue](https://github.com/huaz224/ele-sell/blob/master/src/components/shopcart/shopcart.vue)
line 301-331*


# box-sizing
box-sizing是CSS3的一个新增的属性，主要用于修改默认的盒模型。
## 语法
>box-sizing：content-box | border-box | inherit:

- content-box
默认值，维持了CSS2.1标准盒模型。元素的width和height是指内容区的宽高，不包括padding、border和margin。

- border-box
包括内容，内边距和边框，但不包括外边距。此时，元素的width和height计算如下：
width = border + padding + 内容的width
height = border + padding + 内容的height

- inherit：从父元素继承box-sizing属性的值

## 应用
box-sizing提供了一种更简单的方式，解决响应式布局中存在的问题。
### 解决margin、padding使内容溢出出现滚动条的问题
从一个栗子开始吧：
我们将html和body元素的宽高调整为浏览器窗口的宽高，然后新建一个包裹div元素，它又包含了3个div，高度分别是25%、60%、15%。
//HTML
```
    <div class="wrapper">
        <div class="header"></div>
        <div class="main"></div>
        <div class="footer"></div>
    </div>
```
//CSS
```
    html,body {
        margin: 0;
        height: 100%;
    }
    .wrapper {
        height: 100%;
    }
    .header {
        padding-top: 15px;
        height: 25%;
        background: lightgrey;
    }
    .main {
        height: 60%;
        background: pink;
    }
    .footer {
        height: 15%;
        background: grey;
    }
```

这是我们常见的布局，然而当我们为任意一个div设置margin、padding值时，就会出现高度溢出的问题，比如，当设置header的padding-top为15px时，在纵向上就出现了滚动条，效果：
![](/img/仿饿了么APP商家页面/box.jpg)
这个问题怎么解决呢？？？
我们使用box-sizing属性，将其设置为border-box，然后我们发现：纵向上的滚动条消失了。有没有很嗨森！！！
记得在做商品房网签网备系统时，就经常遇到这个问题，当时是用透明边框来解决的，但是透明边框会影响background属性，显然这种方法的使用会有一定的限制。


### 等高的li列表，避免写margin出现margin重叠的问题
在仿站时，我们需要写很多的li列表，要求它们之间都有相同的间距，如果使用margin就会出现margin重叠的问题（这个稍后会讲哦~^o^~），这里使用box-sizing属性再设置padding值就可以很优雅的实现等高li的效果。
![](/img/仿饿了么APP商家页面/food.png)
源码可戳☛☛☛
*[https://github.com/huaz224/ele-sell/blob/master/src/components/goods/goods.vue](https://github.com/huaz224/ele-sell/blob/master/src/components/goods/goods.vue)
line 209-219*

### 在bootstrap和element ui中的应用
bootstrap和element ui都是栅格式布局系统，它们的实现思路也是一致的：
将整个页面中的一行横向等分为12个格子（element ui是24个），元素的宽度是相对于格子计算的。比如一行中并排包含两个元素，他们的宽度比是1：3，那么在bootstrap中，他们的宽度就是一个占3个格子的宽度，一个占9个格子的宽度。
除了栅格布局系统，其他很多元素也用到了box-sizing这个属性。

我猜想利用box-sizing的原因是：元素的实际宽度只与页面宽度有关，用户手动改变元素padding或border时，不会影响到元素的实际宽度，这样使得布局表现更稳定。

>小贴心：如果不考虑IE6/IE7用户，建议在CSS布局中大胆的使用box-sizing属性，最好可以在 reset（样式重置）的时候就加上它，一劳永逸。。。


# 移动端1px边框
## 1px变胖的原因
一般的，我们在PC端写1px边框都会这样子写：
```
border-top: 1px solid red;
```
那么，在移动端呢？你会发现1px的边框变胖了，它显示的可能是2px或者3px。为什么会出现这样的情况呢？？？
其实这是由于不同的设备具有不同的dpr，dpr是物理像素与逻辑像素在数量上的比值，我们在css中写的1px都是逻辑像素。当手机的dpr=2时，映射到的物理像素就等于2px。
这就是在移动端1px变胖的原因，那么我们如何让1px恢复苗条身材呢？？？hiahia

## 解决方案：使用伪元素
使用的伪元素主要有:after和:before，它们可以在元素之前/后添加内容。
这里实现1px边框的步骤是：
- 给当前元素添加一个伪元素，底部边框添加:after，顶部边框添加:before，content为空；
- 因为伪元素默认为行内元素，转换伪元素的display为block，这样就可以设置它的width和border了；
- 给伪元素绝对定位

我们来看代码：
html
```
    <div class="border-1px">
        移动端1px边框
    </div>
```
css
```
    .border-1px {    
        position: relative;
    }
    .border-1px:after {
        display: block;
        position: absolute;
        left: 0;
        bottom: 0;
        content: ' ';
        width: 100%;
        border-top: 1px solid red;
    }
```
这种方案不仅可以写边框，还可以写圆角，仿站时用的就是伪元素，源码可戳☛☛☛*[https://github.com/huaz224/ele-sell/blob/master/src/common/css/mixin.less](https://github.com/huaz224/ele-sell/blob/master/src/common/css/mixin.less)
line 1-13*
缺点是代码量较大, 占据了伪元素, 容易引起冲突。

# 文本溢出显示省略号的效果
## 单行文本
当文本长度超出一行的宽度时，隐藏溢出文字的效果可以给父容器这样设置：
```
    overflow: hidden;           // 超出隐藏
    white-space: nowrap;        // 不换行
    text-overflow: ellipsis;    // 省略号
```
这里需要注意的是：3个CSS属性是缺一不可。另外，当font-size为0影响省略号的显示时，消除缝隙需要使用其他办法。

*详见header组件，源码请戳☛☛☛
[https://github.com/huaz224/ele-sell/blob/master/src/components/header/header.vue](https://github.com/huaz224/ele-sell/blob/master/src/components/header/header.vue)
line 159-161*

## 多行文本
若要实现多行文本溢出显示省略号，可以使用WebKit的CSS扩展属性，但该方法适用于WebKit浏览器及移动端：
```
overflow: hidden;              // 超出隐藏
display: -webkit-box;          // 设置为弹性伸缩盒子模型显示
-webkit-line-clamp: 2;         // 设置文本最多显示行数
-webkit-box-orient: vertical;  // 设置盒模型的子元素排列方式
```
这里需要注意的是：
- 4个属性也是缺一不可，而且需要设置在文本的父容器上
- WebKit浏览器主要有：Chrome、Safari、搜狗（双核）、Opera（双核）、百度等，兼容多数浏览器
- 设置文本最多显示的行数 = 总高度height / 行高line-height
比如，当文本行高是24px时，如果要在第二行末尾显示省略号，那么总高度是24\*2=48px，如果设置总高度大于48px时，就会在第二行显示省略号，第三行还会显示文字，这显然不是我们要的效果。


# 扩大可点击区域
## 扩大文字a链接的点击区域
本项目在写商品/评论/商家导航栏时，&lt;router-link&gt;被默认渲染为一个&lt;a&gt;标签，我们只有把鼠标移到文字上才能点击导航，这样的用户体验是非常不好的。
为了得到更好的点击体验，我们可以将链接的inline改成block（块），目的是让a自动充满它的父级元素div，这样就点击的范围扩大了。
来看一个栗子：
```
//HTML
	<div class="link">
        <a href="#">商品</a>
    </div>

//CSS 
    .link {
        width: 100px;
        height: 40px;
        line-height: 40px;
        border:1px solid red;
        text-align: center;
    }
    a {
        display: block;
    }   
```
这里需要注意的是：&lt;a&gt;标签是***在宽度上***自动充满它的父级div，如果要在高度上充满，可以设置***&lt;a&gt;标签或者它的父级div的line-height ***。

*详见app组件，源码请戳☛☛☛
[https://github.com/huaz224/ele-sell/blob/master/src/App.vue](https://github.com/huaz224/ele-sell/blob/master/src/App.vue)
line 59*

## 其它普通元素
普通的元素，比如一个小按钮，要在四个方向上扩张它的点击区域可以：
- 增加padding值

- 设置四个方向的透明边框
```
border:6px solid transparent;
```
*详见addcart组件，源码请戳☛☛☛
[https://github.com/huaz224/ele-sell/blob/master/src/components/addcart/addcart.vue](https://github.com/huaz224/ele-sell/blob/master/src/components/addcart/addcart.vue)
line 42*




<- - - 本文 ღ 结束 - - - >