---
title: 仿饿了么APP商家页面-总结Part1（Vue.js）
date: 2017-01-20 23:14:35
categories: 项目总结
tags:
    - vue
    - 仿饿了么APP商家页面
---
最近一个月，利用休息时间做了一个饿了么APP商家页面的仿站。前端使用Vuejs全家桶+ES6+Webpack+Less，后端利用Node+Express模拟了数据。
*（全部项目源码请戳☛☛☛[https://github.com/huaz224/ele-sell](https://github.com/huaz224/ele-sell)，欢迎star）*
这一系列总结包括Vue.js和CSS两个部分的内容，主要来梳理做项目所学到的新芝士，以加深印象。本篇是仿饿了么APP商家页面-总结的第一部分：Vue.js。
<!-- more -->


# ref
ref被用于给元素或子组件注册引用信息。引用都被保存在***$refs对象***中，可以通过“this.$refs.属性”的语法格式来访问。
- 如果是普通元素，引用指向的是***DOM元素***
```
<p ref="p">hello</p>
```
一般来讲，获取DOM元素都要通过document.getElementById等等这样的选择符来获取。这里通过this.$ref.p获取了p标签的引用，减少了dom消耗。
- 如果是子组件，引用指向的是***组件实例***
```
<div class="parent">
  <child-comp ref="child"></child-comp>
</div>

//JS
let child = this.$ref.child;

```
上边的栗子，将对child-comp组件的引用保存在了父组件的$refs对象中。
这里需要注意的是
- 当***ref和v-for一起使用 ***时，不管是普通元素还是子组件，获取到的引用都是一个***数组***。
- ***$refs只有在组件渲染完成后才存在，并且它是非响应式的。***应当在模板挂载之后（mounted）再使用$refs。如果在created中使用，可以结合异步函数this.$nextTick()来使用。

详见goods组件，源码请戳☛☛☛
*[https://github.com/huaz224/ele-sell/blob/master/src/components/goods/goods.vue](https://github.com/huaz224/ele-sell/blob/master/src/components/goods/goods.vue)*


# this.$nextTick
vue更新dom是异步的。也就是说，当你更改了一个响应式数据时，dom并不会立即重新渲染。数据更改触发dom更新的流程是：数据更改-->执行完其它同步命令-->dom更新。因此，在数据更改后试图立即操作更新后的DOM会失败。

为了解决这个问题，我们使用this.$nextTick(callback)，在数据更改后立刻执行其中的回调函数，保证将要获取的dom已经被渲染。

详见goods组件，源码请戳☛☛☛
*[https://github.com/huaz224/ele-sell/blob/master/src/components/goods/goods.vue](https://github.com/huaz224/ele-sell/blob/master/src/components/goods/goods.vue)
line 74*

# 路由的重定向
『重定向』的意思是当用户访问/a时，URL将会被替换成/b。通过routes配置来完成。
来看下面的例子：
```
export default new Router({ 
  routes: [{
      path: '/a',
      redirect: '/b'
    },{
      path: '/a',
      redirect: { name: 'foo' }
    }]
})
```
重定向的目标可以是字符串路径或者命名的路由对象，甚至是一个方法，动态返回重定向目标。
在本项目中，我们利用重定向实现了将路由视图初始化为商品展示页面。
```
import goods from '@components/goods/goods'

export default new Router({ 
  routes: [{
      path: '/',
      redirect: 'goods' 
    }]
})

```
源码请戳☛☛☛
*[https://github.com/huaz224/ele-sell/blob/master/src/router/index.js](https://github.com/huaz224/ele-sell/blob/master/src/router/index.js)
line 13*

# 配置router-link链接激活时的CSS类名
&lt;router-link&gt;支持在路由中（点击）导航，可以通过to属性指定目标地址，它会被默认渲染为一个&lt;a&gt;标签。我们知道，一般的&lt;a&gt;标签通过伪类选择器:active来设置链接激活时的类名。
```
a:active{ //注意这里是:号
	background-color:yellow;
}
```
那么，&lt;router-link&gt;如何配置链接激活时的CSS类名呢？？？
有两种方法：
- 方法一：默认值: "router-link-active"
在CSS中需要这样写：
```
a { 
	&.router-link-active{} //注意这里是.号
}
```
  每次都写这么长的类名，懒病是不是又犯了，，，
  Vue为我们提供了对默认值的全局配置：可以在路由的构造选项中，通过linkActiveClass属性来自定义一个CSS类名。
  ```
  export default new Router({
    linkActiveClass: 'active', 
    routes: []
  })
  ```
  这酱紫，就可以在任何一个文件中通过类名active来设置链接激活时的样式了。
- 方法二：可以通过active-class属性来自定义链接激活时的类名。
```
<router-link to="/goods" active-class="active">商品</router-link>
```
这种方法的缺点就是：在每个&lt;router-link&gt;标签都需要定义active-class属性，显然这种方法适合标签数量较少时。

# 过度动画
## 什么是过度动画？
开始我们先啰嗦一下，什么是过度动画？
一个元素以什么形式出现，中间变化了什么，并且以什么形式消失，是有一个过渡的存在的方式，这个变化的过程就是过度动画。

Vue过度动画本质是利用CSS3的过度和3D转换，我们需要先了解一下，教程请戳☛☛☛
- [CSS3 过渡： transition属性](http://www.w3school.com.cn/css3/css3_transition.asp)
- [CSS3 动画： @keyframes规则、animation属性](http://www.w3school.com.cn/css3/css3_animation.asp)
- [CSS3 transform 属性](http://www.w3school.com.cn/cssref/pr_transform.asp)

这里要记住几个单词：transition是过度，transform是转换，translate是移动，rotate是旋转，scale是缩放。

## 过度的语法和情形
那么，在vue中怎么写过度呢？？？
vue提供给我们一个很好写过渡的内置组件transition，只要用&lt;transition&gt;标签将进行过度的元素包裹起来，并给上属性，起码name不要忘了。
语法：
```
<transition name="move">
    <!-- 将要进行动画的元素 -->
</transition>
```
哪些元素可以进行过度呢？不管是普通元素，还是组件，都有四种情形可以进行过度：
- 条件渲染 (使用 v-if)
- 条件展示 (使用 v-show)
- 动态组件
- 组件根节点

## 过度的类名
Vue给过度配置了6个类名：
- v-enter
定义进入过度的开始状态，只在一帧间存在，在下一帧移除。
- v-enter-active
定义过度的状态，在一段距离上存在，在transition/animation完成之后被删除。
- v-enter-to
定义进入过度的结束状态，此时v-enter被删除，在一段距离上存在，在transition/animation完成之后被删除。
- v-leave
定义离开过度的开始状态，只在一帧间存在，在下一帧移除。
- v-leave-active
定义过度的状态，在一段距离上存在，在transition/animation完成之后被删除。
- v-leave-to
定义离开过度的结束状态，在一段距离上存在，此时v-leave 被删除，在transition/animation完成之后移除。

图示说明：
![](/img/仿饿了么APP商家页面/transition.jpg)
这里需要注意的是：
- v-enter 和 v-enter-active 同时生效的，前者在下一帧移除，后者在动画完成后移除。
- v-enter 和 v-leave都只在一帧间存在，其它4个class都发生在一段距离上。

栗子：
```
.fade-enter-active,.fade-leave-active{
  transition: all 0.5s ease-out;
}
.fade-enter{
  transform: translateY(-500px);
  opacity: 0;
}
.fade-leave-active{
  transform: translateY(500px);
  opacity: 0;
}
```
以上代码定义的动画是这样子的：过度总时长0.5秒，并且以ease-out曲线形式从Y轴-500px移动到+500px，并且伴随着一个透明度的变化：开始是0，最终状态是1，过度结束后又变回0。

## 应用
在仿站时，针对购物车添加/删除商品定义一个动画：用0.4s以linear曲线形式在X轴方向位移24px，这个过程伴随着180度的旋转。
源码请戳☛☛☛
*[https://github.com/huaz224/ele-sell/blob/master/src/components/addcart/addcart.vue](https://github.com/huaz224/ele-sell/blob/master/src/components/addcart/addcart.vue)
line 46-58*
 


<- - - 本文 ღ 结束 - - - >