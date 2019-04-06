---
title: 框架篇之react-router
date: 2018-04-22 20:55:55
categories: 读书笔记
tags:
- JavaScript
- 深入理解JS
---
***框架篇之react-router***：本篇主要总结SPA单页面以及在react中怎么使用路由来跳转页面。


<!--more-->

什么时候用路由？同一个区域展示不同的内容

# SPA
1. SPA与MPA
SPA：（single page）单页面应用，只有一个页面，所有需要展示的内容都在这一个页面中切换，webpack中只需要配置一个入口（移动端居多 或者  PC端管理系统类也是单页面应用为主）。
MPA：（multi page web application）多页面应用，一个项目由很多页面组成，使用这个产品，主要是页面之间的跳转（PC端多页面应用居多）：基于框架开发时，需要在webpack中配置多入口，每一个入口对应一个页面。
2. 如何实现单页面应用？
1）如果项目是基于服务器渲染的，后台语言中可以基于include等技术，把很多部分拼凑在一起，实现组件化或插件化开发，也可以实现单页面应用。
2）基于iframe实现单页面应用（父页面可以嵌入子页面，一个iframe通过修改的src切换不同的子页面）。
3）模块化开发：
AMD：require.js
CMD：sea.js
基于这些思想把每一部分单独写成一个模块，最后基于grunt/gulp/fis等自动化工具，最后把所有的模块都合并到首页面中（包括HTML、CSS、JS都合并在一起），通过控制哪些模块的显示隐藏来实现单页面应用开发。
弊端：由于首页中的内容包含了 所有模块的信息，所以第一次加载速度很慢。
4）基于vue、react的路由实现SPA单页面应用，基于webpack打包等。

3. BrowserRouter和HashRouter
- BrowserRouter：浏览器路由
是基于H5的history API（pushState，replaceState，popState）来保持UI和URL的同步，真实项目中应用的不多。一般只有当前项目是基于服务器端渲染的，才会使用浏览器路由。

- HashRouter：哈希路由
一般前后端分离的项目使用哈希路由。一个HTML页面通过不同的HASH值呈现不同的组件。它基于原生JS构造了一套类似于history API的机制。每一次路由的切换都是基于history stack（历史栈）完成的。
    + 当前项目一旦使用HashRouter，则默认在页面的地址后面加'#/'，也就是HASH默认值是一个斜杠，一般让其显示首页内容
    + HashRouter中`只有一个子元素`
    + 根据哈希地址不同，展示不同的组件内容，此时需要使用Route组件

# Route组件
Route组件，可以设置一些属性：
- path：匹配哈希后面的值
匹配规则：页面的哈希值必须完全包含完整path的值（不多也不少），`非严格匹配`。Route`不设置path`是匹配所有URL地址。
例如：
path='/' ：和它匹配的地址只有要斜杠即可（都能和它匹配）
path='/user'：“#/user/login”也可以匹配，但是“#/user2”这个无法匹配


- `exact`：path匹配严格（只有URL的hash值和path设定的值相等才可以匹配到）
strict也是严格匹配，不常用
- component：哈希值和当前Route的path相同，则渲染component指定的组件
- render：当页面的hash地址和path匹配，会把render规划的方法执行。
    一般在render中`处理权限校验`。（在组件渲染之前，没有权限不渲染这个组件）。
- push：如果设置了这个属性，当前跳转的地址会加入到history stack中一条记录
- from：设置当前来源页面地址
```
import {HashRouter, Route, Switch, Redirect} from 'react-router-dom';
//=>A B C 三个组件
ReactDOM.render(<HashRouter>
    <Switch>//如果可以匹配多个，Switch只匹配第一个
        <Route path='/' exact component={A}/>
        <Route path='/user' component={B}/>{/*#/user/login*/}
        <Route path='/pay' component={C} render={() => {
            let flag = localStorage.getItem('FLAG');
            if (flag && flag === 'safe') {
                return <C/>;
            }
            return '当前环境不安全，不利于支付！';
        }}/>
    </Switch>
</HashRouter>, root);
```
默认情况下，会和每一个Route都做校验（哪怕之前已经有校验成功的）。Switch组件可以解决这个问题，只要有一种情况校验成功，就不再往后校验了。


以上都不符合的情况下，认为路由地址是非法地址，做一些特殊处理
- 处理为404
- 也可以重定向：
        to=[string]： 重新定向到新的地址
        to=[object]：
         {
          pathName: 定向的地址,
            search: 给定向的地址问号传参,（真实项目，会根据是否存在问号参数值来判断是正常进入首页还是非正常跳转过来的，也有根据问号传参值做不同的事情）
            state: 给定向后的组件传递一些信息
            } 
```
    <Switch>
        {/*处理404*/}
        <Route render={() => {
            return <div>404</div>
        }}/>
        {/*也可以重定向：to重新定向到新的地址*/}
        <Redirect to='/'/>
    <Redirect to={{
        pathname: '/',
        search: '?lx=404'
    }}/>
    <Redirect from='/360buy' to='/jd'/>
    </Switch>
```

# Link和NavLink组件
1. Link组件是react-router中提供的路由切换组件，基于它可以实现`点击时切换路由`。
react-router中提供的组件都要在`任何一个Router（常用HashRouter）包裹的范围内`使用。
- to[string] ：跳转到指定的路由地址
- to[object] ：可以提供一些参数配置项
{
pathname：跳转地址
serach：问号传参
state：基于这种方式传递信息值
}
- replace[boolean]：默认为false，是替换history stack中当前的地址（true），还是追加一个新的地址（false）

原理：基于Link组件渲染，渲染后的结果就是a标签，to对应的信息最后会变为href中的内容
```
<Link className='navbar-brand' to={{
                    pathname: '/',
                    search: '?lx=logo'
                }}>我的CRM</Link>
<a class="navbar-brand" href="#/?lx=logo">my CRM</a>
```

2. NavLink和Link类似，都是为了实现路由切换跳转的。
=>
不同的在于：NavLink组件在`当前页面URL的HASH地址和组件to对应地址相匹配`时，会默认给组件加一个active样式类，让其有选中态。并不是点击谁就有active类名。
和Link类似，to和replace属性都有，用法一样
- activeClassName：把默认加的active样式类改为自己的
- activeStyle：给匹配的这个NavLink设置行内样式
- exact & strict：控制是否是严格匹配
- isActive：匹配后执行对应的函数
```
<NavLink to='/custom'>//最后也会转换为a标签，如果当前页面的hash地址和此组件中的to地址匹配了，则会给渲染后的A标签设置默认的样式类：active
```
所有经过路由管控的组件，当URL的hash和path后面的地址匹配后，组件才会重新渲染。不经路由管控的组件，点击NavLink，不会重新渲染组件。

# withRouter组件
作用：把一个非路由管控的组件，模拟成为路由管控的组件。
先把Nav基于connect高阶一下，返回的是一个代理组件Proxy，把返回的代理组件受路由管控即可。
```
import {withRouter} from 'react-router-dom';
export default withRouter(connect()(Nav));
```


# history stack
受路由管控组件的特点：
- 只有当前页面的哈希地址和路由指定的地址匹配，才会把对应的组件渲染。
withRouter比较特殊：如果没有地址匹配，会被模拟为受路由管控的
- 路由切换的原理：凡是匹配的路由，都会把对应的组件内容重新添加到页面中，相反，不匹配的都会在页面中移除掉，下一次重新匹配上，组件需要重新渲染到页面中：每一次路由切换时（页面的哈希路由地址改变），都会从一级路由开始重新校验一遍。
- 所有受路由管控的组件，在组件的属性上，都默认添加了3个属性：    
  + history：
    - go()：跳转到指定的地址（数字：0-当前，-1-上一个，-2上两个）
    - push()：向池子中追加一条新的信息，达到切换到指定的路由地址的目的。
    - goBack()：相当于go(-1)，回退到上一个地址
    - goForward()：相当于go(1)，向前走一步
  + location：获取当前哈希路由渲染组件的一些信息 
    - pathname：当前哈希路由地址
    - search：当前页面的问号传参值
    - state：基于Redirect/Link/NavLink中的to，传递的是一个对象，对象中编写的state，就可以在location.state中获取到
  + match：获取的是当前路由匹配的结果
    - params：获取的是当前路由匹配的是`地址路径参数`，则这里可以获取传递参数的值
方法：this.popps.history.push('/plan')

history stack：历史信息栈（池子）
每一次路由的切换，不是替换现有的地址，就是新增一条地址。

`基于render返回的组件是不受路由管控的组件`，没有这三个属性。

# `【列表详情模型】`
在SPA路由管控的项目当中，从列表跳转到详情，需要传递一些信息给详情组件，以此来展示不同的信息，传递给详情页信息的方式有：
[不推荐的]
- 本地存储
- redux存储
=>点击列表中某一项时，把信息存储到本地或redux中，跳转到详情页面，把信息从本地或redux中获取即可

[推荐]
- `问号传参`（随便刷新）
```
    //list页
    <Link to={{
        pathname: '/custom/detail',
        search:`?id=${id}`
    }}>
        //...
    </Link>
    //detail页
    let search = this.props.location.search,//'?id=1'
        curID = qs.parse(search.substr(1)).id || 0;
    curID = parseFloat(curID);
```
- `基于state传值`（弊端：一旦页面刷新，上一次传递的state值就没有了）
适合支付页面。
```
    //list页
    <Link to={{
        pathname: '/custom/detail',
        state:`${id}`
    }}>
        //...
    </Link>
    //detail页
    let state = this.props.location.state,//'1'
        curID = state || 0;
    curID = parseFloat(curID);
```
- `URL地址参数`：（把参数当做地址的一部分）
path = 'custom/detail/:id'
第一步：在路由中配置动态路径参数
```
  //custom页
  <Route path='/custom/detail/:id' component={Detail}/>
```
第二步：
```
  //list页
    <Link to={{
        //pathname: '/custom/detail',
        pathname: `/custom/detail/${id}`,//3.URL地址参数
    }}>
        //...
    </Link>
```
第三步：this.props.match.params.id来获取
```
  //detail页
  let params = this.props.match.params,//{id: "34"}
```


<- - - 本文 ღ 结束 - - - >