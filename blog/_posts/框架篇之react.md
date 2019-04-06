---
title: 框架篇之react
date: 2018-04-07 23:35:05
categories: 读书笔记
tags:
- JavaScript
- 深入理解JS
---
***框架篇之react***：本篇博文，主要用来总结在项目中用到的React的基础知识，包括脚手架配置、JSX语法、渲染机制、钩子函数、函数式组件和声明类式组件、组件传参等等，以加深印象。


<!--more-->

# React介绍
1. React是一个MVC框架。特点：
- 划分组件开发
- 基于路由的SPA单页面开发
- 基于ES6写代码（最后部署上线时，需要把ES6编译成ES5，基于Babel来完成编译）
- 可能用到less/sass等，也需要使用对应的插件进行预编译
- 最后为了优化性能（减少HTTP请求次数），需要把JS或者CSS进行合并压缩
webpack完成以上页面组件合并、JS/CSS编译加合并等工作。



2. React全家桶：react / react-dom / react-router / redux / react-redux / axios / ant / dva / saga / mobx

3. react有两部分组成：
- react：框架的核心部分，提供了Component类进行组件开发，提供了钩子函数（生命周期）
所有的生命周期函数都是基于回调函数完成的。
- react-dom：把JSX语法（react独有的语法）渲染为真实DOM的组件（能够放到页面中展示的结构都叫做真实的DOM）

# 脚手架

## 安装create-react-app
脚手架：一个插件，基于它可以快速构建一套完整的自动化工程项目结构。
Vue：vue-cli
React：create-react-app（app：应用）
- 安装脚手架
安装在全局环境下（目的：可以使用命令）
```
 npm install create-react-app -g
```

## 项目新建、启动、打包
- 新建一个项目：
项目命名规范：`小写字母、数字、下划线、中划线`，不能出现大写字母、中文汉字、其它特殊符号等，（和npm发包时的命名规范一样）
```
create-react-app [项目名称]
```
- 如果是克隆的别人的项目，直接安装依赖项
```
npm install 或 yarn install 
```

- 启动一个项目
```
npm run start 或 yarn start
```
开发环境下，基于webpack编译处理，最后可以预览当前开发的项目成果（在webpack中安装了webpack-dev-server插件，基于这个插件会自动创建一个web服务[端口号是3000]），webpack会帮我们自动打开浏览器，展示页面并且监听页面，实现热更新。

- 打包
```
npm run build 或 yarn build
```
将项目整体编译打包的命令，（打包会生成build文件夹，包含所有编译后的内容，上传到服务器即可）


## 项目目录结构
- node_modules：当前项目安装的依赖包
	+ .bin：本地项目可执行命令，在package.json的scripts中配置对应的脚本即可（其中一个就是react-scripts命令）
-  `public`：存放的是当前项目的`HTML页面`
	单页面放一个index.html，现在项目大多是单页面。
	index.html中一般只有一个root标签，其它所有自己写的代码都是写在JS中的。
- src：存放的是所有的JS、路由、组件等（包括需要编写的css或图片等）
	+ `index.js`是当前项目的`主入口文件`
- .gitignore：git提交时的忽略文件目录
- packjson.json：当前项目的配置清单
create-react-app脚手架为了让结构目录清晰，把安装的webpack及配置文件都集成在了react-scripts模块中，放到了node_modules中。
	+ `react-scripts`：集成了webpack需要的所有内容
babel一套
css处理一套
eslint一套
webpack一套
=>没有less/sass的处理内容（项目中使用less，需要自己额外安装）
```json
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
```


## 修改默认配置项
真实项目在安装脚手架的基础上，需要额外安装一些模块，例如：react-router-dom / axios，再比如：less / less-loader...
情况一：安装其它的组件成功后，不需要修改webpack的配置项，此时我们直接安装，并且调取使用即可。
情况二：安装的插件是基于webpack处理的，也就是需要把安装的模块配置到webpack中。

1. 开启HTTPS协议模式（原理：设置环境变量HTTPS的值）
```
set HTTPS=true&&yarn start
```
原理：在webpackDevServer.config.js中，修改环境变量HTTPS的值
```javascript
const protocol = process.env.HTTPS === 'true' ? 'https' : 'http';
```
2. 更改端口号
```
set PORT=63344&&yarn start
```
原理：在scripts / start.js中，修改DEFAULT_PORT的值（可以直接修改环境变量PORT，也可以直接修改后边的3000）
```javascript
const DEFAULT_PORT = parseInt(process.env.PORT, 10) || 3000;
```
3. eject操作：把node_modules中隐藏的配置项暴露到项目中（不可逆）
```
yarn eject
```
提示确认是否执行，如果当前项目是基于git管理，在执行eject时，如果还有没有提交到历史区的内容，需要先提交到历史区，然后再eject，否则报错。
一旦暴露后，项目中多了两个文件夹config和scripts
- config：存放的是webpack的配置文件
	+ webpack.config.dev.js：开发环境下的配置项（yarn start）
	+ webpack.config.prod.js：生产环境下的配置项（yarn build）
	+ webpackDevServer.config.js：配置创建端口号为3000的服务，在浏览器预览
- scripts：存放的是可执行脚本的JS文件
	+ start.js：yarn start执行的JS文件
	+ build.js：yarn build执行的JS问价
package.json文件也被修改了。

开发环境和生产环境，如何在webpack中区分环境？
node中的process.env用于设置环境变量，通过不同的环境变量（development和production），规定哪个环境下执行的操作（不同的环境执行不同的配置文件）。

4. 支持less
预览项目，也是先基于webpack编译，把编译后的内容放到浏览器中运行。如果项目中使用了less，需要修改webpack配置项，在配置项中加入less的编译工作，这样后期预来项目，会首先基于webpack对less文件进行编译。
```
yarn add less less-loader
```
less是需要开发和生产环境都需要配置的，所以安装在全局。
开发环境下：在webpack.config.dev.js中
```javascript
	//=>[DEV:159~193行]
  {
    test: /\.(css|less)$/,//=>增加less
    use: [
      //...
      {
            loader: require.resolve('less-loader'),//=>增加less-loader
      },
    ],
  }
```
生产环境下：
```javascript
    {
        test: /\.(css|less)$/,
        loader: ExtractTextPlugin.extract(
            Object.assign(
                {
                    //...
                    use: [
                        //...
                        {
                            loader: require.resolve('less-loader'),
                        }
                    ],
                }
            )
        ),
    }
```


## 一些细节问题
一个/：根目录
在react中，所有的逻辑都是在JS中完成的（包括页面结构的创建），如果想给当前页面导入一些CSS样式或IMG图片等内容，有两种方式：
- 在JS中（JS导入的资源都会基于webpack编译）
基于·ES6 Module规范，import导入·（或者·CommonJS规范，require导入·）
=>这样webpack在编译合并JS时，会把导入的资源文件等插入到页面的结构中（`绝对不能`在JS管控的结构中通过`相对目录./或../导入资源`，因为在webpack编译时，地址就不再是之前的相对地址了）
例如，在目录src / index.js中：
```javascript
/*公共的样式在在index中导入*/
import './static/css/reset.min.css';
import './static/images/1.jpg';
```
- 在HTML中导入
HTML最后也要基于webpack编译，导入地址也`不建议`写相对地址，而是使用`%PUBLIC_URL%写成绝对地址`
例如，在目录public / index.html中：    
```vbscript-html
    <link rel="stylesheet" href="%PUBLIC_URL%/css/reset.min.css">
```




# JSX语法
JSX：`JavaScript + XML（HTML）`
JSX可以创建一个 `React 元素`。每一个JSX只能有一个根元素。

和之前拼接的HTML字符串类似，都是把HTML结构代码和JS代码或者数据混合在一起，但它不是字符串。

1. `JSX中的{}`：存放JS代码，可以将数据嵌入到JSX中。
- 引用类型的值：
	+ 不能直接放对象（除了给style赋值）
	+ 数组（数组中不能有对象，基本值或JSX元素可以）
	+ 函数不行（`自执行函数可以`）
```javascript
  ReactDOM.render(<div>
      <h2>{{name:'xxx'}}</h2>     NO
      <h3>{new Date()}</h3>       NO
      <h3>{[12,23,34]}</h3>       OK
      <h4>{(() => {
          return '呵呵';          OK：嵌入自执行函数的结果
      })()}</h4>
    <p>
        {(function some() {
            console.log(1);      OK：不管是否有返回值
        })()}
    </p>
  </div>, root);
```
- 基本类型的值
	+ 布尔类型：什么都不显示
	+ null / undefined：也是JSX元素，代表的是空

- JS表达式：但是要求执行完有`返回的结果`
	+ 使用三元运算符解决判断操作，（if和switch都不可以）	
	+ 循环数组创建JSX元素（一般都是基于数组的map方法完成迭代），需要给创建的元素设置唯一的key值（当前本次循环内唯一即可）
```javascript
  //data：数组，存放的是对象（包含name和age两个属性）
  ReactDOM.render(<ul>
      {
          data.map((item, index) => {
	          let {name, age} = item;
              return <li key={index}>
                  {name：}&nbsp;&nbsp;{age}
              </li>;
          })
      }
  </ul>, root);
```
2. 给JSX元素设置属性：
属性值对应大括号中`对象、函数都可以放`（也可以放JS表达式）
style属性值必须是对象（不能是样式字符串）
class用className代替


# 渲染
把JSX（虚拟DOM）变为真实的DOM。
JSX渲染机制：
1. 第一步，基于babel中的语法解析模块（babel-preset-react）把JSX语法编译为React.createElement(...)
babel是一个强大的正则解析库。

2. 第二步，执行React.createElement(type, props, children)，创建一个对象（这个对象就是虚拟DOM）
这个对象的属性有
	+ type：标签名
	+ props：一个对象，存放的是这个标签上的属性
		+ id
		+ className
		+ style
		+ children：存放的是元素中的内容
	+ ref：
	+ key
	+ ...
	+ \__proto__：Object.prototype
3. 第三步，基于render方法，把动态生成的虚拟DOM元素，插入到指定的容器中
ReactDOM.render([JSX],[container],[callback]);
	+ JSX：`React虚拟元素`
	+ container：容器，把元素放到页面中的那个容器中，不建议把JSX直接渲染到document.body中，一般挂载到一个ID为root的div中（根节点）。可以基于document.getElementById('root')获取，也可以直接写root。
	+ callback：把内容放到页面中呈现触发的回调函数
```javascript
ReactDOM.render(<div>
    //...
</div>, root);
```






# 钩子函数
生命周期函数：
描述一个组件或程序从创建到销毁的过程，在这个过程中，基于钩子函数完成一些操作（例如，在第一次渲染完成做什么，或者第二次即将重新渲染之前做什么。。。）

```
static defaultProps = {};//=>这个是第一个执行的，执行完成后（给属性设置默认值后）才向下执行
```
1. 基本流程：
- `constructor`：创建一个组件
传props，给状态初始化
- `componentWillMount`：第一次渲染之前
在这获取DOM元素是undefined。相当于Vue的beforeMount。
- `render`：第一次渲染
- `componentDidMount`：第一次渲染之后
DOM元素挂载完成，可以获取到。相当于Vue的mounted。
=>
真实项目中，在DidMount这个阶段做如些处理：
- 控制状态更改的操作
- 从服务器获取数据，然后`修改状态信息`，完成`数据绑定`

注意：
在WillMount中，
如果直接setState修改数据，状态数据改变后，执行render和DidMount（`最新的state值`）。有一个小坑：==this.setState本身就是异步的，会通知执行render==，`WillMount中this.state的值还是原来的`。
如果将this.setState放在一个异步操作中（定时器和异步AJAX获取数据）完成，先执行render和DidMount，然后才异步更改状态数据，因为异步任务都要等第一次tick后（DidMount执行完）才会执行。
=>
真实项目中的数据绑定，一般第一次组件渲染，绑定默认的数据，第二次才是绑定的从服务器获取的数据（有些要求我们需要根据数据是否存在判断显示隐藏等）


2. 修改流程：
2.1 【属性更新触发】
- `componentWillReceiveProps(nextProps, nextState)`：父组件调用子组件时，传递的属性发生改变后触发
==三种情况会触发==：
		- 父组件通过props传递的数据发生改变
		- redux中的数据发生变化
		- 路由切换引起的history/match/location变化会触发
这个方法执行完成，会接着执行should这一套修改的流程。
2.2【状态更新触发】
当`组件的状态数据发生改变`（setState）或者`传递给组件的属性发生改变`（重新调用组件传递不同的属性），都会引发render重新进行渲染（差异渲染）
- `shouldComponentUpdate(nextProps, nextState)`：是否允许重新渲染（返回true允许，则执行后面的钩子函数，不允许直接结束）
        方法中this.state.xxx获取的还是更新前的状态信息，但should方法有两个参数：
        nextProps：最新修改的属性值
        nextState：最新修改的状态信息
=>
适合做一些拦截，防止多次重新渲染，消耗性能。
- `componentWillUpdate(nextProps, nextState)`：重新渲染之后
这里拿到的this.state.xxx也是更新前的数据，和should一样，也有两个参数
- `render`：第二次及以后重新渲染
- `componentDidUpdate`：重新渲染之后
3. 卸载
- `componentWillUnmount`：卸载组件之前（一般不用）
卸载并不是销毁组件，原有渲染的内容是不消失的，只不过以后不能基于数据改变视图了。


![Alt text](./QQ图片20180621123000.png)

[vue与react类（对）比学习总结](https://www.jianshu.com/p/ba8c59be80b6)

注意：this.state.n++
```javascript
this.setState({
	n: this.state.n++
});
```
注意：this.state.n++是先执行this.state.n（异步），然后再++（同步），假如n初始值为0，同步++先执行，变为1，主栈空闲时执行异步代码，又变为0。


# 组件
按照组件/模块管理的方式来构建程序，也就是把一个程序划分为一个个的组件来单独处理。
优势：多人协作开发，组件被复用。
组件的存放目录：src->component

react创建组件有两种方式：
函数声明式和基于Component类创建组件

## 函数声明式
1. 基础语法：
- 每一个组件都要导入一个react，因为需要基于它的createElement把JSX进行解析渲染。
- 函数返回结果是一个新的React元素（也就是当前组件的JSX结构）
- `props`变量存储的是一个`对象`，包含了调取组件时传递的属性值（不传递是一个空对象）
		- children：（单闭合和双闭合组件标签区别）
			+ 组件没有子元素时，是undefined
			+ 组件有子元素时，可只是一个值或一个数组，可能每一项是一个字符串，也可能是一个`对象`等（`{}中可以放react自己生成的对象`，比如props.children）
```javascript
import React from 'react';
export default function Dialog(props) {
    let {con, lx} = props;
    let title = lx === 1 ? '系统提示' : '系统警告';
    return <section>
        <h2>{title}</h2>
        <div>{con}</div>
        {props.children}
        {
            React.Children.map(props.children, item => item)
        }
    </section>;
}
```
React中提供了`Children对象`，对象中的map、forEach等方法专门用于遍历props.children。

2. 使用：
在JSX中调取组件：只需要把组件当做一个标签使用（单闭合、双闭合都可以）
传递给组件的属性值：不是字符串的，要放在{}中
```
ReactDOM.render(<div>
    <Dialog/>
    <Dialog con='哈哈' lx={1}>
        <span>1</span>
    </Dialog>
</div>, root);
```

3. 渲染机制
createElement在处理时，遇到一个组件，type就不再是字符串标签名，而是一个函数（类），但是属性还是存在props中的。
```
{
	type:Dialog,//=>组件Dialog
	props:{
		lx:1,
		con:'xxx',
		children:一个值或一个数组
}
```

==render渲染时，需要做处理==：
- 首先判断type的类型，如果是字符串，就创建一个标签元素，如果是函数或者类，就`把函数执行，把props中的每一项（包含children）传递给函数`。
- 在执行函数时，把函数中return的JSX转换为新对象（通过createElement），然后把这个对象返回。紧接着render按照以往的渲染方式，创建DOM元素，插入到指定的容器中。


## 继承类式（Component类）
1. 两种创建类方式，在渲染机制上的区别：
基于createElement把JSX转换为一个对象，当render渲染这个对象时，遇到type是一个函数或者类，不是直接创建元素，而是先把方法执行：
- 函数声明式组件：就把它当做`普通方法执行`（严格模式下，方法中的this是`undefined`），把函数返回的JSX元素（也是解析后的对象）进行渲染。
- 类声明式组件：把`当前类new执行`，创建类的一个实例（当前本次调取的组件就是它的一个实例），执行constructor之后，会执行this.render()，把render中返回的JSX拿过来渲染。=>类声明式组件，`必须有一个render的方法`，方法中需要`返回一个JSX元素`（返回12也可以）。


不管是哪种方式，最后都会把解析出来的`props属性对象作为实参`传递给对应的函数或者类。



2. super()相当于`React.Component.call(this)`
function Component(props, context, updater){...}
`super()`，虽然创建实例时把属性传递进来了，但是并没有传递给父组件，也就是没有把属性挂载到实例上，使用this.props获取的结果是undefined
`super(props)`，在继承父类私有时，就把传递的属性挂载到了子类的实例上，constructor就可以使用this.props了。

即使在constructor中不设置形参props接收属性，执行super时也不传这个属性，除了constructor中不能直接使用this.props，`其它生命周期函数中都可以使用`（也就是执行完成constructor，react已经帮我们把传递的属性接收，并且挂载到实例上了）。


## 应用
函数声明式：
- 操作简单
- 能实现的功能，只是简单的调取和返回JSX而已
- 函数式组件可以理解为`静态组件`（组件的内容调取是就已经固定了，很难修改）
所谓函数式组件是静态组件：和执行普通函数一样，调取一次组件，就把组件中的内容获取到，插入到页面中。如果不重新调取组件，显示的内容是不会发生任何改变的。
适用于：`调取一次组件，以后组件中的内容不会再次改变的情况`，例如，获取数据，绑定HTML
```
function Clock() {
    return <section>
        <h3>当前北京时间是：</h3>
        <div style={{fontSize: '20px', lineHeight: '2'}}>
            {new Date().toLocaleString()}
        </div>
    </section>;
}

setInterval(() => {
    ReactDOM.render(<Clock/>, root);
}, 1000);
```


继承类式：
- 操作相对复杂一些，但是可以实现更为复杂的业务功能
- 能够使用生命周期函数操作业务
- 继承类式，可以基于组件内部的状态（state）来动态更新渲染组件的情况
```
class Clock extends React.Component {
    constructor() {
        super();
        //=>初始化组件的状态
        this.state = {
            time: new Date().toLocaleString()
        };
    }
    componentDidMount() {
        setInterval(() => {
            this.setState({//异步操作
                time:new Date().toLocaleString()
            },()=>{
            });
        }, 1000);
    }
    render() {
        return <section>
            <h3>当前北京时间是：</h3>
            <div style={{fontSize: '20px', lineHeight: '2'}}>
                {this.state.time}
            </div>
        </section>;
    }
}
ReactDOM.render(<Clock/>, root);
```

## 属性（props）和状态（state）
1. props
属性（props）：`[只读]`，调取组件时传递进来的信息。
函数或类的方法来声明组件，都有props属性，都无法修其自身 props。

props是只读的，我们无法在方法中修改它的值，但是可以给其设置默认值或者设置一些规则
```
    static defaultProps = {
        lx: '系统提示'
    };
```
安装prop-types插件：给组件传递的属性设置规则（设置的规则不会影响组件渲染，但会在控制台抛出警告错误）。
```
    static propTypes = {
        // con:PropTypes.string,//=>传递的内容是字符串
        con: PropTypes.string.isRequired,//=>传递的内容不仅是字符串，并且必须传递
    };
```



2. state
状态（state）：`[读写]`，自己在组件中设定和规划的（`只有类声明式组件`才有状态管控，函数式组件没有）
- 初始化组件的状态为一个`对象`：在constructor中把后期需要使用的状态全部初始化一下。
```javascript
    constructor() {
        super();
        this.state = {
            //...
        };
    }
```
- 初始化之后，可以通过this.state来获取。
- 基于setState修改组件状态（Component.prototype.setState）
`异步操作`
```
Component.prototype.setState = function (partialState, callback){
	//=>callback：当部分状态修改，组件把对应的部分元素渲染完成执行的回调函数
}
```
1）修改部分状态：只对该初始化的状态对象的一部分属性或全部属性，修改哪个写哪个
2）当状态修改完成，会通知react把组件JSX中的部分元素重新进行渲染。
组件状态类似于Vue中的数据data，数据绑定时，是基于状态值绑定的，当修改组件状态后，对应的JSX元素也会跟着重新渲染（差异渲染：只把数据改变的部分重新渲染，基于DOM-DIFF算法完成）。



3. 区别

props和state的区别：
state是组件的私有数据，props是父组件传递过来的
props是只读的
state可以调用this.setState修改state值（不可以直接修改this.state！）
state代表的是子组件自身的内部状态。从语义上讲，改变组件的状态，可能会导致dom结构的改变或者重新渲染。
无论是state改变，还是父组件传递的 props改变，render方法都可能会被执行。 

框架最重要的核心思想就是：数据操控视图（视图影响数据），告别JQ手动操作DOM的时代。
在react中，
- 基于数据驱动（修改状态数据，react会重新渲染视图）完成的组件叫做受控组件（受数据控制的组件）
- 基于ref操作DOM实现视图更新的，叫做非受控组件
=>真实项目建议使用受控组件

react：[MVC] 数据更改视图跟着更改（原本是单向数据绑定）
但是可以构建出双向的效果。
使用change事件

## 复合组件传参


>父组件传递数据给子组件：
基于属性传递（传递是单方向的）：
1）父组件中：在子组件标签上添加属性
2）子组件中：通过this.props.[属性名]获取传递的信息

子组件中的数据需要修改：可以让父组件传递给子组件的信息发生变化（也就是调用子组件时传递的属性发生变化，子组件会重新渲染=>触发componentWillReceiveProps钩子函数）

>子组件修改父组件的状态：
1）把父组件的一个方法，作为属性传递给子组件
2）在子组件中，执行基于属性传递过来的方法。（相当于在执行父组件中的方法：而这个方法完全可以操作父组件中的信息）


# 组件传参
## 复合组件
复合组件：父组件嵌套子组件
父传子
1. 属性传递（props）：
调取子组件时，把信息基于属性的方式传递给子组件（子组件用props接收传递的信息）（只能父传子，不能子传父，基于属性传递信息是单向传递的）


2. 上下文传递（context）
父组件先把需要传给后代元素（包括孙子元素）的信息都设置好（设置在上下文中）后代组件需要用到父组件中的信息，主动去父组件中调取使用即可。
- 在父组件中：
1）设置子组件上下文属性值类型
```javascript
	 //=>父组件设置信息
	 static childContextTypes = {
	     //=>设置上下文中信息值的类型
	     n: PropTypes.number,
	     m: PropTypes.number
	 };
```   
2）获取（设置）子组件的上下文，返回什么就传递给子组件什么
```javascript
	 getChildContext() {
	     //->RETURN的是啥，相当相当于往上下文中放了啥
	     let {count: {n = 0, m = 0}} = this.props;
	     return {
	         n,
	         m
	     };
	 }
```
注意：`只要RENDER重新渲染，就会执行getChildContext这个方法`，重新更新父组件中的上下文信息（render=>context=>子组件调取渲染）；如果父组件上下文信息更改了，子组件在重新调取的时候，会使用最新的上下文信息；   
- 在子组件中：
设置传递进来的上下文类型：设置哪个类型，子组件context才有哪个类型
this.context.xxx
```javascript
     //=>子组件主动获取需要的信息
     static contextTypes = {
         //=>首先类型需要和设置时候类型一样，否则报错；并且你需要用啥，就写啥即可；
         n: PropTypes.number,
         m: PropTypes.number
     };
```
子组件怎么获取：`this.context.[父组件的上下文暴露出来的]`
3. 属性 VS 上下文
- 属性操作简单，子组件被动接收传递的值（组件内的属性是只读的），只能父传子（子传父不行，父传孙也需要：父传子，子再传孙）
- 上下文操作起来相对复杂，子组件是主动获取信息使用的，子组件可以修改获取到的上下文信息，但是不会影响到父组件中的信息，其它组件也不受影响。
一旦父组件设置了上下文信息，它的后代组件都可以直接拿来用，不需要一层层传递。
=>
【其实，子组件可以修改父组件的信息】
利用回调函数机制：父组件把一个函数通过属性或上下文的方式传递给子组件，子组件中只要把这个方法执行即可（也就是子组件中执行了父组件方法，还可以传递一些值过去），这样父组件在这个方法中，想把自己的信息改成啥就改成什么。


## 平行组件
平行组件：兄弟组件或毫无关系的两个组件
- 方案一：让两个平行组件有一个共同的父组件
父：Parent
子：A / B
父组件中的信息，父组件把信息传递给A，A中把方法执行（方法执行修改父组件信息值），父组件再把最新的信息传递给B即可，等价于A操作，影响了B。
- 方案二：基于redux进行状态管理，实现组件之间的信息传输（常用方案）



组件在什么情况下会重新渲染：
组件状态变化
属性变化
上下文变化


<- - - 本文 ღ 结束 - - - >