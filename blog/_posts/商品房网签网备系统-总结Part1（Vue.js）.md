---
title: 商品房网签网备系统-总结Part1（Vue.js）
date: 2017-09-16 22:48:34
categories: 项目总结
tags:
- vue
- ElementUI
- 商品房网签网备系统
---

从二月份到现在的几个月里，收货非常的大：我和公司的一个前端小姐姐（江湖称号：叁哥）一起完成了商品房网签网备系统的雏形，作为演示demo得到了领导的认可。在此要特别感谢叁哥，之前连Vue是什么都不知道，是她告诉我原来有Vue这么神奇的框架，人生难遇领路人呢~^o^~。本项目是我学习前端以来第一个实战项目，要求做PC端和移动端APP。前端使用Vue全家桶（vue + vuex + vue router + vue resource + elementUI + Less），后端采用C#调用WebService提供数据服务（内网服务）（感谢后端哥哥的大力支持~^o^~）。
*（全部项目源码请戳☛☛☛[https://github.com/huaz224/realEstate](https://github.com/huaz224/realEstate)，欢迎star）*
这一系列总结主要想整理的是，项目所用到的新方法、思路、以及遇到的坑，包括Vue、HTML5、CSS、ES6四个部分。本篇是商品房网签网备系统-总结的第一部分：Vue2.x，主要从Vue的环境配置到具体的使用来详细地进行总结。
<!--more-->


# 环境搭建
安装Vue有两种方法：
- 直接&lt;script&gt;引入CDN
```
<script src="https://unpkg.com/vue"></script>
```
- 通过脚手架vue-cli创建vue项目（需要使用npm来安装）
CLI是Vue官方提供的一个命令行工具，需要对node.js有一定的了解，不推荐新手直接使用。

本项目采用第二种方法。

## 安装node.js
进入【[node.js官网](https://nodejs.org/en/download/)】，下载对应版本，一路傻瓜式安装即可。
安装完成后，cmd打开命令行输入 node -v 和 npm -v，如果能显示出版本号，就说明安装成功。
>小贴士：如果卸载重新安装，傻瓜式卸载后还需删除C:\Users\CXH\AppData\Roaming目录下的npm和npm-cache两个文件夹。（因为这个被折腾好久哦）

## 安装vue-cli
在命令行输入
```
npm install -g vue-cli
```


## 新建项目
新项目是基于webpack模板创建的。通过命令行窗口cd进入即将要新建项目的目录，然后输入
```
vue init webpack my-project 
```
执行后，按照提示手动输入项目名称、描述、作者、打包方式、是否安装vue-router（yes，这个用的很多哦）、是否使用ESLint检查代码（no）、是否建立单元测试（no）等内容后，会在当前目录生成一个my-project的项目文件夹。
接下来，cd进入刚才新建的项目文件夹，安装依赖
```
npm install
```
安装完成后，启动项目

```
npm run dev
```

>小贴士：如果你安装了Git，右键git bash打开黑窗口，所有的命令都可以直接输入，不用再cmd、cd啦。

>小贴士：启动已有项目只要npm install-->npm run dev两步就OK辣，依赖只是在第一次启动时需要install。

这时会自动启动chrome，并跳转到localhost:8080页面（因为CLI带有热重载，Ctrl+S后会自动更新到浏览器辣）。

PS：看到Vue帅帅的界面，说明环境搭建成功，可以开始熟悉项目结构，写组件了哦。。。

## 可选的扩展
Vue官方提供了多种扩展，我们可以根据需要进行选择。本项目使用了vue-router、vue-resource、vuex来辅助开发。
- vue-router，用于配置路由
输入以下命令进行安装，（新建项目时安装了vue-router略过此处）
```
npm install vue-router --save
```
    具体的使用请戳官方文档☛☛☛[vue-router](https://router.vuejs.org/zh-cn/index.html)
- vue-resource，处理HTTP请求，与服务端通信
输入以下命令进行安装，
```
npm install vue-resource --save
```
    具体的使用请戳官方文档☛☛☛[vue-resource](https://www.npmjs.com/package/vue-resource)
- vuex，全局状态管理，其实就是一个存数据的
输入以下命令进行安装，
```
npm install vuex --save-dev
```
    具体的使用请戳官方文档☛☛☛[Vuex](https://vuex.vuejs.org/zh-cn/)



# 入门介绍
## 项目结构
项目的目录结构如图所示。我们需要关注的是config目录、src目录、static目录和index.html。

|目录名|子目录/子文件|描述|
|--------|---------|---------|
|bulid文件夹|dev-server.js为开发的webpack打包的入口文件 |  |
|config文件夹|index.js |项目配置文件，包括端口号等的配置|
|src文件夹是项目源码目录，其中初始包括三个文件夹和两个文件|assets文件夹 |一般存放图片等资源，会被webpack构建|
| |components文件夹 |存放以.vue为后缀的组件|
| |router文件夹 |其中的index.js用于配置路由|
| |App.vue |默认根组件，也就是其他所有组件的父组件，可以把整个页面看成是一个Vue的大组件|
| |main.js |入口js文件，主要作用是初始化vue实例并使用需要的插件|
|static文件夹|是纯静态资源文件夹，其中的文件不会被webpack构建|   |
| .babelrc  | 配置babel ，将Vue中的ES6语法编译为ES5|  |
|index.html|入口页面|  |
|package.json|项目配置文件，包含项目描述信息、项目生产环境和开发环境所需的依赖| |

## 组件结构
一个标准的vue组件，包含三个部分：template、script、css。下面分别来介绍这三部分应该注意的问题。
- template
template标签是html元素的书写区域，这里有一个坑，一个组件下**有且仅有一个**孩子节点，**不一定是div**，也可以是html别的元素。
- script
script标签是js的书写区域。一般在&lt;/script&gt;标签的第一行import引入vue组件或者js文件，在export default {}里以“属性名：选项属性”的格式写data、methods、computed、watch、钩子函数等选项。
```
<script>
import login from './components/login'
import { mapState, mapMutations } from 'vuex'

export default {
  name: 'app',
  data() {
    return {}
  },
  components: {
    login
  },
  computed: {},
  watch: {},
  methods: {},
  created: function() {}
}
</script>
```
    这里需要注意的是： 
     可以在**&lt;/script&gt;标签的第一行**引入import…from引入**其他vue组件**或**js文件**。如果引入了组件，引入之后要注册再使用；如果是js文件，可在methods中直接使用（**不用加this哦**，因为并不是Vue实例的方法）。
    ```
    <script>
    import login from './components/login'
    import leftNav from './components/leftNav'
    import { strToObj } from '../../js/generalMethods.js'
    export default {
      components: {//与data并列
        login,
        leftNav
      },
      methods: {
        //可以直接调用strToObj
      }
    }
    </script>
    ```

- css
css标签是CSS样式的书写区域。需要注意的是：
 - style标签添加“scoped”属性，是用来限定样式的作用域的，也就是所有样式只能被当前组件使用；而如果不加scoped属性，那么这个标签里的样式可供**全局**使用。
 - 如果使用less语法，需要添加lang=‘less’属性


## data
数据要写在**data函数return的对象里**，而不是直接放在data里。注意，data函数是ES6写法，等同于data:function(){}
```
<script>
export default {
  data() {
    return {
      a: "hello",
      b: 1
    }
  }
}
</script>
```
注意：只有在data中声明的数据才是响应式的。在data之外无法创建一个根级别的响应式数据。


## methods
方法要写在**methods对象**里，并且是**“键值对”**的格式，键是方法名，值是函数，有没有嗅到函数表达式的味道？
```
<script>
export default {
  methods: {//与data并列
    show: function(){
      //函数体
    }
  }
}
</script>
```
注意：值不能是箭头函数，但在值内部可以使用箭头函数。

## computed 与 watch
都是用来观察和响应Vue实例上的数据变化的。
- computed属性
一个computed属性是**依赖于一个或几个data中的数据“计算”**得到的。需要注意的是：
 - 计算属性的结果会被缓存，除非依赖的data数据发生了变化，否则不会重新计算。
 - 依赖是响应式的，非响应式的不会触发计算属性更新。这块理解的还不是恨透彻，，，
```
computed: {
  now: function () {
    return Date.now();//Date.now()不是响应式依赖
  }
}
```

- watch
watch属性是一个观察者，用来“监视”相应数据是否变化。
```
<script>
export default {
  name: 'statList',
  data() {
    return {
      statData: [], //table绑定的数组对象
      isShow: false
    }
  },
  watch: {
    statData: function(val, oldVal) {
      if (val.length !== 0) {
        this.isShow = true;
        //当table绑定的数组不为空时，显示表格
      }
    }
  }
}
</script>
```
    *源码请戳☛☛☛
    [https://github.com/huaz224/realEstate/blob/master/src/components/gov/statList.vue](https://github.com/huaz224/realEstate/blob/master/src/components/gov/statList.vue)
    line 152-158*

- 区别
computed属性是要**返回**一个依赖于某个数据变化而变化的**数据**，返回的这个数据可以像data中的普通数据一样在template中使用，而watch属性是要在**发现某个数据变化后，dosomething**。


## 钩子函数
每个Vue实例在被创建时都要经过一系列的初始化过程，同时在这个过程中也会运行一些叫做生命周期钩子的函数。这里需要注意的是：
- 钩子的this指向调用它的Vue实例
- 不要在选项属性或回调上使用箭头函数

**常用的钩子函数包括：**
- beforeCreate
vue实例创建后立即执行，此时data、methods都没有被初始化，无法直接访问。
- created
数据、事件等初始化完毕执行，但DOM并未生成，el属性不存在。
可以直接访问data和methods等，但无法获取DOM元素。
这个钩子里比较适合写Loading结束、http请求、一些初始化操作。
*（本项目使用vue-resource在created钩子里写了http请求，详见query、modeMng等组件。）*
- beforeMount
模板挂载之前，完成了el属性的初始化，但并未挂载，仅仅是“占坑”。节点是虚拟的DOM节点。比如，data中有一个message属性，占坑还是显示的是{{message}}，并没有显示实际的值。
- mounted
模板挂载之后，el属性被新创建的vm.$el替换，模板中的html渲染到了页面中。
*（可参考本项目showMode组件。元素挂载后，如果是只读模式则所有Input只读。源码请戳☛☛☛[https://github.com/huaz224/realEstate/blob/master/src/components/inc/showMode.vue](https://github.com/huaz224/realEstate/blob/master/src/components/inc/showMode.vue)  line 33-41）*
- beforeUpdate
数据更新之前执行，虚拟DOM需要重新渲染和打补丁。
- updates
数据更新之后执行


## 条件渲染 v-if 与 v-show
- v-if
v-if是一个根据条件来进行渲染的指令，可以搭配v-else、v-else-if来控制多个div的显示隐藏，条件可以是**条件表达式**，也可以是**某个对象的属性**，会自动转换为Boolean值。
**应用**
一般用户登录之后显示主要内容界面，就不再显示登录页。在本项目中，通过v-if和v-else控制登录页和主要内容的切换显示，代码如下：
```
<template>
  <div id="app">
    <div class="login" v-if="!user.name">
      <!-- !user.name根据state状态的变化转换为false -->
      <login></login>
    </div>
    <div class="content" v-else>
    </div>
  </div>
</template>
```
    *源码请戳☛☛☛[https://github.com/huaz224/realEstate/blob/master/src/components/login.vue](https://github.com/huaz224/realEstate/blob/master/src/components/login.vue)
    更多应用可参考本项目leftNav、userMng组件。*

- v-show
另一个根据条件来渲染的指令就是v-show，但v-show只是简单地***切换元素的CSS属性 display***。不同的是带有v-show的元素始终会被渲染并保留在DOM中。
```
<template>
  <div id="userMng">
    <el-button type="primary" v-show="isShow">下一步</el-button>
  </div>
</template>
<script>
export default {
  name: 'userMng',
  data() {
    return {
      isShow: true
    }
  }
</script>
```
- 区别
 - v-if 是“真正”的条件渲染，v-show仅仅是基于CSS进行切换
 - v-if在条件为假时，不去渲染元素，直到条件第一次变为真才会开始渲染；v-show则始终渲染
 - v-if可以写在template标签内控制整个模板的渲染与否，v-show不可以，只能写在template标签的子元素中
 - v-if 可以与v-else、v-else-if联合使用；v-show不可以
 
 综上，v-if适用于运行时条件不太可能改变的场景，比如：异步请求数据，在数据返回之后渲染页面时使用v-if更好；v-show适合可能频繁切换元素的显示和隐藏的场景，比如hover。


# ElementUI
ElementUI是饿了么团队推出的一款UI框架，它的布局、配色等等是非常简洁优美的。本项目主要界面都是用这个搭的哦，个人非常稀罕~^o^~
- 需要先安装
```
npm i element-ui -S
```
- 在入口main.js文件中完整引入ElementUI和css样式，并使用Vue.use()明确的安装ElementUI
```
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-default/index.css'
Vue.use(ElementUI);
```
这样就可以使用ElementUI快速的搭建美美的页面辣，官方文档请戳☛☛☛[Element - 网站快速成型工具](http://element.eleme.io/#/zh-CN)

# Less
Less是一种CSS预处理语言，赋予了CSS如变量、继承、运算、函数等动态语言的特性。
要在服务端使用，需要输入以下命令进行安装less和less-loader
```
npm install less less-loader --save-dev
```
然后在style标签添加lang=‘less’属性就可以使用啦
官方文档请戳☛☛☛[Less 中文网](http://lesscss.cn/)

# 打包项目
项目开发完成后，进入项目目录输入
```
npm run build
```
将项目打包，打包成功后会生成dist文件夹，这个文件夹里的文件就是最终的成果，它部署到服务器上就OK辣。


# 遇到的问题
## 组件之间传参的几种方式
### props down, events up
在 Vue 中，父子组件的关系可以总结为 props down, events up。父组件通过 props 向下传递数据给子组件，子组件通过 events 给父组件发送消息。
- props
父组件向子组件传数据时，可以在父组件的HTML标签添加中间属性（注意是v-band:中间属性='data里的属性名'），然后子组件中使用props注册该中间属性（注意：props:['xxx']加引号，注册位置与data并列）后，即可在子组件中使用来自父组件的数据

- 使用$emit触发事件传递数据
 - 当子组件的方法被调用时，子组件通过this.$emit('自定义事件名称',this.子组件data中的属性)触发自定义事件
 - 父组件在子组件HTML标签里直接用 v-on 来监听自定义事件的触发，进而触发父组件自己的事件
 - 在父组件自己的事件中，定义时传入一个参数，这个参数赋值给父组件data里的属性，就完成了传参。

这种传参方式在项目中用的并不多，就不再啰嗦了。

### 使用路由实例的push方法
**push方法介绍**
在Vue实例内部，你可以通过$router 访问路由实例。父路由可以通过push()方法，导航到不同的URL。push()方法的参数可以是一个字符串路径，也可以是一个描述地址的对象。
```
// 字符串
router.push('home')
// 对象
router.push({ path: 'home' })
// 命名的路由
router.push({ name: 'user', params: { userId: 123 }})
```

>注意：如果提供了path，params会被忽略。

```
// 这里的params不会生效
router.push({ path: '/user', params: { userId: 123 }})
```

**项目中的应用**
在本项目中，使用命名的路由在两个组件之间传参。父路由通过在name属性指定向哪个路由传参数，通过params属性将要传递的参数放在一个对象中。在这个对象中，参数以“参数名：参数值”的格式传递，参数名可以自己随便取，就像声明变量一样；多个参数以","隔开。
```
this.$router.push({name: 'showMode',params:{ id: 44}});
```
子路由通过 this.$route.params.参数名 来接收父路由传递过来的参数
```
this.$route.params.id
```
这样传参有一个问题：刷新就消失。因此，非常有必要学习HTML5的web本地存储。

### 使用HTML5的web存储
详见[本系列总结第二部分-HTML5的web存储](https://huaz224.github.io/2017/09/23/%E5%95%86%E5%93%81%E6%88%BF%E7%BD%91%E7%AD%BE%E7%BD%91%E5%A4%87%E7%B3%BB%E7%BB%9F-%E6%80%BB%E7%BB%93Part2%EF%BC%88HTML5%EF%BC%89/#more)

## 数组或对象的更新检测问题
对于数组，受JS限制，Vue不能检测data之外数组的以下变动：
 - 当你利用索引直接更改一个项时，arr[index] = newValue;
 - 当修改数组的长度时，arr.length = newLength;

同样地，Vue无法动态地创建一个根级别的响应式属性。

>可以通过Vue.set(target, key, value)或arr.splice()解决，注意vm.$set只是全局Vue.set的别名

**栗子：**
```
export default {
  data() {
    return {
      arr: [0, 1],
      person: {
        name: 'chen'
      }
    }
  },
  methods: {
    add: function() {
      //数组
      this.arr[0] = 'a';
      this.arr.length = 4;
      this.$set(this.arr, 2, "c");
      //对象      
      this.$set(this.person, "age", 24);
      this.person["sex"] = "男";
    }
  }
}
```
//结果
![](/img/商品房网签网备系统/data.jpg)

>小贴士：this.$set()方法调用时，页面会全部渲染一遍，所以没有使用Vue.set()方法所做的更改也更新到了DOM，比如this.arr.length = 4;

*在本项目中，houseData对象是响应式的，但添加属性后非响应式的，并不会渲染DOM。
详见query组件，源码请戳☛☛☛
[https://github.com/huaz224/realEstate/blob/master/src/components/inc/query.vue](https://github.com/huaz224/realEstate/blob/master/src/components/inc/query.vue)
line 133*


# 最后想说
通过这个项目，我深深的体验到了Vue的强大，它抛弃了传统的直接操作dom的习惯，而是采用data数据驱动view的理念，使新手如我都可以快速上手。学到这里，只是入了Vue的门，要想精通其实还有很长的路要走，一起加油辣。
最后，奉上Vue全家桶全部参考文档：
- [vue.js](https://cn.vuejs.org/v2/guide/)
- [vue-router](https://router.vuejs.org/zh-cn/index.html)
- [Vuex](https://vuex.vuejs.org/zh-cn/)
- [vue-resource](https://www.npmjs.com/package/vue-resource)
- [一起玩转Vue-resource](http://www.jianshu.com/p/3ce2bd36596e)
- [npm使用入门](http://www.jianshu.com/p/4643a8e43b79)
- [Element - 网站快速成型工具](http://element.eleme.io/#/zh-CN)
- [Less 中文网](http://lesscss.cn/)
- [LESS « 一种动态样式语言](http://www.bootcss.com/p/lesscss/)

<- - - 本文 ღ 结束 - - - >