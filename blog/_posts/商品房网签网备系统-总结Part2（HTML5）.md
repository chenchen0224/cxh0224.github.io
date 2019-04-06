---
title: 商品房网签网备系统-总结Part2（HTML5）
date: 2017-09-23 19:26:03
categories: 项目总结
tags:
- HTML
- 商品房网签网备系统
---
本篇是商品房网签网备系统-总结的第二部分：HTML5，主要来总结项目中所用到的HTML5特性，包括&lt;footer&gt;标签、类扩展classList、web 存储等等。
<!--more-->
# footer标签
&lt;footer&gt;标签常用来定义页脚，属于block元素。
栗子：
```
<footer>
  CopyRight&nbsp;©&nbsp;2017&nbsp;XXX开发有限公司
</footer>
```

# 类扩展classList
HTML5为所有元素添加了classList属性，用于操作类名。这个classList是新集合类型DOMTokenList的实例，包含以下属性和方法：
- 属性：length 表示包含多少个元素
- 方法：
item()：取得每个元素，或者使用[]
add(value)：添加给定的字符串，若已存在则不添加。
remove(value)：从列表中删除字符串，若不存在则不删除。
contains(value)：列表中是否包含字符串，返回true/false。
toggle(value)：若列表中已存在字符串，则删除；不存在，则添加

这样一来，只用一行就可删除一个类名：
比如：
```
<div class="a b c"></div>
```
```
div.classList.remove("b");
```
需要注意的是，这些方法一次仅能操作一个元素。
有了classList属性，除非完全重写或者删除类名，一般也用不到className了。因为className属性获得的类名是一个字符串，增删改特别麻烦。
# web 存储
## localStorage和sessionStorage
HTML5 web 存储，是一个比cookie更好的本地存储方式，包括localStorage和sessionStorage两个存储对象。
- localStorage对象
localStorage是一个**没有时间限制的**数据存储。也就是说，一旦存储，永远不会删除，这么占内存，肯定用的不多了。
- sessionStorage 对象
sessionStorage是HTML5提出的一个针对session 数据存储的对象。数据以“键/值”对的形式存储，**生命周期是当前窗口**。前进后退刷新，数据都存在；关闭窗口，删除数据。

## 常用API
不管是 localStorage，还是 sessionStorage，可使用的API都相同，常用的有如下几个（以sessionStorage为例）：
  - 保存数据：sessionStorage.setItem(key,value);
  - 读取数据：sessionStorage.getItem(key);
  - 删除单个数据：sessionStorage.removeItem(key);
  - 删除所有数据：sessionStorage.clear();
  - 得到某个索引的key：sessionStorage.key(index);

## 在项目中的应用
主要用于在两个组件之间传参，即使刷新页面还可以获取到参数。

>有人说组件传参用props或者store啊，其实这里有一个坑。刷新一下页面，看看还能获取到吗？结果是，props传过来的获取不到，store存储的刷新之后为初始值。

*在本项目中，statList组件和developer组件之间传参，需要将用户选择的开发商名称developers_name保存，developer组件读取数据，从而根据developers_name生成当前表格数据。*
```
sessionStorage.setItem("devName",JSON.stringify(row.developers_name)); 
```
*源码请戳☛☛☛[https://github.com/huaz224/realEstate/blob/master/src/components/gov/statList.vue](https://github.com/huaz224/realEstate/blob/master/src/components/gov/statList.vue)  line 195*
```
data () {
   return {
     //通过JSON.parse将sessionStorage存储的数据解析为js值
     developer: JSON.parse(sessionStorage.getItem("devName")),
   }
}
```
*源码请戳☛☛☛[https://github.com/huaz224/realEstate/blob/master/src/components/gov/developer.vue](https://github.com/huaz224/realEstate/blob/master/src/components/gov/developer.vue)  line 39*

<- - - 本文 ღ 结束 - - - >