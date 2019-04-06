---
title: sticky-footer
date: 2017-12-06 21:22:39
categories: 技术水波文
tags:
- CSS
---
我们所写大部分静态页面，很多时候都会把一个页面分为头部、内容和页脚。当页面高度小于一屏高度时，footer要固定到底部，而当页面超出一屏高度时，footer要随着高度走。这种技术有一个非常洋气的名字：sticky-footer。
<!--more-->

有四种方案实现sticky-footer布局。

# 方案一：相对定位和margin负值
适用于：footer块高度已知
html
```
    <div class="wrapper-add">
        <div class="content">
            <p>start</p>
            <!-- 此处省略n个p标签-->
            <p>end</p>
        </div>
    </div>
    <footer>
        CopyRight&nbsp;©&nbsp;2017&nbsp;XXX有限公司
    </footer>
```
css
```
    html,
    body,
    p {
        height: 100%;
        margin: 0;
    }
    .wrapper-add {
        min-height: 100%;
    }
    .content {
        padding-bottom: 25px;/*等于footer高度*/
        background-color: pink;
    }
    footer {
        position: relative;
        margin-top: -25px;/*等于content的padding高度*/
        height: 25px;/*等于content的padding高度*/
        clear: both;
        background-color: grey;
    }
```
另外，为了保证兼容性，需要在wrapper上添加clearfix类。其代码如下：
```
.clearfix{
     display: inline-block;
}
.clearfix:after {
     content: ".";
     display: block;
     height: 0;
     clear: both;
     visibility: hidden;
}  
```
这种负margin的布局方式，是兼容性最佳的布局方案，各大浏览器均可完美兼容，适合各种场景。缺点是：前提是必须要知道footer元素的高度，且会改变HTML结构，代码复杂。

# 方案二：绝对定位和padding-bottom
比较适用于footer块高度已知时，主要设置包裹div的min-height为100%和footer块的绝对定位。代码如下：
html
```
    <div class="wrapper-add">
        <div class="content">
            start<br>
            <!-- 此处省略n个换行符-->
            end<br>
        </div>
        <footer>
            CopyRight&nbsp;©&nbsp;2017&nbsp;XXX有限公司
        </footer>
    </div>
```
css
```
html,
body {
    height: 100%;
    margin: 0;
}

.wrapper-add{
    position: relative;
    min-height: 100%;
}

.content {
    padding-bottom: 25px;/*等于footer高度*/
    background-color: pink;
}

footer {
    position: absolute;
    bottom: 0;    
    height: 25px;/*等于content的padding高度*/
    width: 100%;
    background-color: grey;
}
```

这里实现的思路是：
对footer进行绝对定位，并且设置bottom:0，让footer固定在父容器的底部。因为footer绝对定位是根据父容器的实际高度来的，因此我们要保证父容器高度在不足一屏时设置为一屏，在超过一屏时，高度为实际高度，通过min-height就可以实现了。另外，还要保证兄弟容器的padding-bottom值等于footer的高度值，footer容器就可以占据兄弟容器的padding位置，两个兄弟的高度正好等于父容器的100%高度。这样就OK了。
有几点需要解释一下：
- wrapper-add容器设置min-height 为100%，而不是height:100%，这很关键，可以使它在内容很少（或没有内容）的情况下，能保持一屏的高度。ie6不识别min-height,可做如下处理
```
height: auto !important;
height: 100%;
```
- content设置padding-bottom为20px，并让该值等于footer的高度，这也非常关键
- footer容器必须设置一个固定高度，单位可以是px(或em)

这里也可以使用box-sizing：
```
.wrapper-add{
    position: relative;
    min-height: 100%;
    padding-bottom: 25px;/*等于footer高度*/
    box-sizing: border-box;
    background-color: pink;
}

footer {
    position: absolute;
    bottom: 0;    
    height: 25px;/*等于main的padding高度*/
    width: 100%;
    background-color: grey;
}
```

# 方案三：flex布局
flex布局方式非常简洁，不仅html结构简单，而且footer块高度未知也适用。
从一个栗子开始，先来看html部分
```
    <div class="wrapper-add">
        <div class="content">
            <p>start</p>
            <!-- 此处省略n个p标签-->
            <p>end</p>
        </div>
        <footer>
            CopyRight&nbsp;©&nbsp;2017&nbsp;XXX有限公司
        </footer>
    </div>
```
css
```
    html,
    body {
        margin: 0;
        height: 100%;
    }
    .wrapper-add {
        display: flex;
        flex-flow: column;
        min-height: 100%;
    }
    .content {
        flex: 1;
    }
    footer {
        flex: 0;
    }
```
这里的实现思路是：
1）给内容和页脚加一个包裹div，将这个div设置为弹性容器，主轴为垂直方向；
2）包裹div设置min-height为100%，或者height:100%，这很关键，可以使它在内容很少（或没有内容）的情况下，能保持一屏的高度；
3）给内容设置flex为1；
4）给footer设置flex为0，如果要设置高度，可以设置flex:0 0 25px;


# 方案四：calc()函数
你认识calc()函数吗？这货其实就是一个计算长度值的。那么用calc()怎么来实现sticky-footer呢？？？
非常简单，我们来看代码
html
```
    <div class="content">
        <p>start</p>
        <!-- 此处省略n个p标签-->
        <p>end</p>
    </div>
    <footer>
        CopyRight&nbsp;©&nbsp;2017&nbsp;XXX有限公司
    </footer>
```
css
```
    html,
    body,
    p {
        height: 100%;
        margin: 0;
    }
    .content {
        min-height: calc(100% - 25px);/*减去footer高度*/
        background-color: pink;
    }
    footer {
        height: 25px;
        background-color: grey;
    }
```
这种代码极其精简，适合于布局简单的情况。


# 总结
不管采用哪种方案，效果见图：
当页面高度没超过一屏时，见图
![](/img/sticky-footer/小于等于.jpg)
当页面高度超出一屏高度时，见图
![](/img/sticky-footer/大于.jpg)

<- - - 本文 ღ 结束 - - - >