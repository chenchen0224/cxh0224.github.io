---
title: 框架篇之redux
date: 2018-05-17 23:58:55
categories: 读书笔记
tags:
- JavaScript
- 深入理解JS
---
***框架篇之redux***：本篇主要来总结redux的基础流程，以及在工程化项目中，是如何来进行全局状态管理的。


<!--more-->
# 什么是redux
redux：统一状态管理的一个类库
应用场景：
- 只要两个或者多个组件的信息需要共享，把共享信息放到redux容器中
- 临时存储：页面加载时，把从服务器中获取的数据存储到redux中，组件渲染需要的数据，从redux中获取，这样只要页面不刷新，路由切换时，再次渲染组件不需要重新从服务器拉取数据，直接从redux中获取即可；页面刷新，从头开始。
（redux代替了localStorage本地存储来实现数据缓存）


redux是`临时存储`，页面一旦刷新，所有组件和程序重新运行，之前存储的redux信息会消失。


# 基础流程
redux可以应用在任何项目中，vue、react、JQ等都可以。
react-redux专门提供给react。
redux提供了以下API：
```
export {
  createStore,
  combineReducers,
  bindActionCreators,
  applyMiddleware,
  compose,
  __DO_NOT_USE__ActionTypes
}
```
1. 创建一个存储状态的容器（对象）：需要把reducer传递进来
```
import {createStore} from 'redux';
let store = createStore(reducer);
```
源码：
```
createStore(reducer, preloadedState, enhancer) {
    //...
    return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  }
}
```
创建的store对象中提供了三个方法：
- dispatch：派发行为（传递一个`对象`，对象中有一个type属性），通知reducer修改状态信息
- subscribe：事件池追加方法
- getState：获取最新管理的状态信息


2. reducer：是一个`函数`，类似于自己写的callback
[作用]
- 记录了所有状态修改的信息（根据行为标识走不同的修改任务）
- 修改容器中的状态信息

[参数]
- state：容器中原有的状态信息（如果第一次使用，没有原有状态，给一个初始默认值）
- action：`dispatch任务派发的时传递的行为对象`（必有一个`type属性`，是操作的行为标识，reducer就是根据这个行为标识来识别该如何修改状态信息）
```
let reducer = (state = {n: 78, m: 100}, action) => {
    switch (action.type) {
        case 'VOTE_SUPPORT':
            state = {...state, n: state.n + 1};
            break;
        case 'VOTE_AGAINST':
            state = {...state, m: state.m + 1};
            break;
    }
    return state;//=>只有把最新的STATE返回，原有的状态才会被修改
};
```


使用：
VoteFooter.js：
初始化状态：获取redux中的状态数据
```javascript
    let reduxState = this.props.store.getState();
    this.state = {...reduxState};//{n,m}
```
真实项目中，会把redux容器中的状态信息获取到，赋值给组件的私有状态或者是属性（react-redux），目的是：当redux中的状态改变，可以修改组件内部的状态，从而达到重新渲染组件的状态。

向发布订阅事件池中追加一个方法，监听redux的状态，当状态更改时，会通知事件池把追加的方法执行，重新渲染组件：
```javascript
    componentDidMount() {
        let {store: {getState, subscribe}} = this.props;
        let unsubscribe = subscribe(() => {//=>名字可任意
            let {n, m} = getState();
            this.setState({
                n,
                m
            });
        });
        //unsubscribe(); 把当前追加的方法移除，解除绑定的方式
    }
```
VoteFooter.js：通过`dispatch派发任务`（同步任务），修改容器状态
```javascript
    let {store: {dispatch}} = this.props;
    dispatch({
        type: 'VOTE_AGAINST'
    });
```

# redux的工程化结构
1. 全局配置
  store：
- reducer：存放每一个模块的reducer
       vote.js
       personal.js个人中心
       ...
       index.js 把每一个模块的reducer最后合并成为一个reducer
 
- action：存放每一个模块需要进行的派发任务（ActionCreator）
       vote.js
       personal.js
       ...
       index.js  所有模块的action进行合并
 
- action-types.js：管控当前项目中所有redux任务派发中需要的行为标识，定义为常量
- index.js  创建store


`action-types.js`
```
export const VOTE_SUPPORT = 'VOTE_SUPPORT';
export const VOTE_AGAINST = 'VOTE_AGAINST';
```

一个项目中只有一个reducer，reducer / index.js每一个模块的reducer最后合并成为一个reducer。
`reducer / index.js`
```
import {combineReducers} from 'redux';
import vote from './vote';
import personal from './personal';
let reducer = combineReducers({
    vote,
    xxx: personal
});
export default reducer;
```
保证合并每个模块管理的状态信息不会相互冲突：
redux在合并的时候把容器中的状态进行分开管理（以`合并reducer时设置的属性名`做为状态划分的属性名，把各个板块管理的状态放到自己的属性下即可）
```
    STATE={
       vote:{
         n:0,
         m:0
       },
      xxx:{
        baseInfo:{}
      }
  }
```
以后获取状态信息的时候，也需要把VOTE等模块名加上
store.getState().vote.n


`index.js`
```
import {createStore} from 'redux';
import reducer from './reducer/index';//<=>'./reducer/index'

let store = createStore(reducer);
export default store;
```




# react-redux
react-redux是把redux进一步封装，适配react项目。
在组件调取使用时，可以优化一些步骤：

## Provider：根组件
当前整个项目都在Provider组件下，作用就是把创建的store可以供内部任何后代组件使用。（基于上下文完成的）
- Provider`只有一个子元素`
- 只需要把创建的store，基于属性传递给Provider（后代就都可以使用这个store了）。
```
ReactDOM.render(<Provider>
    //=>只允许出现一个子元素
</Provider>, root);
```

## connect：高阶组件
- 导出的不再是创建的组件了，而是基于connect构造后的高阶组件。（删掉组件前面的export default）
语法：`export default connect([mapStateToProps], [mapDispatchToProps])([自己创建的组件])`
```javascript
//...=>自己创建的类
//=>把redux容器中的状态信息遍历，赋值给当前组件的属性（state）
let mapStateToProps = state => {
    //=>state：就是redux中的状态信息
    //=>返回的是啥，就把啥挂载到当前组件的属性上
    return {
        ...state.todo
    };
}
//=>遍历redux中的dispatch派发行为，也赋值给组件的属性（ActionCreator）
let mapDispatchToProps = dispatch => {
    //=>dispatch：store中存储的dispatch方法
    return {
        add(payLoad) {
            dispatch(action.todo.add(payLoad));
        }
        //...
    };
}
export default connect(mapStateToProps, mapDispatchToProps)([自己创建的组件])
```
可以进一步简化代码：
```JavaScript
//...=>自己创建的类
export default connect(state =>({...state.todo}), action.todo)([自己创建的组件]);//=>为什么这么写可以：react-redux把ActionCreator中编写的方法（返回action对象的方法），自动构建成dispatch派发任务的方法，也就是mapDispatchToProps这种格式
```
- 基于subscribe向事件池追加方法，一达到容器状态信息改变，执行我们追加的方法，重新渲染组件的目的，但是现在不用了，react-redux帮我们做了这件事：所有用到redux容器状态信息的组件，都会向事件池中追加一个方法，当状态信息改变，通知方法执行，把最新的状态信息作为属性传递给组件，组件值重新改变了，组件就会重新渲染了。

这样配置之后，所有基于connect挂载到当前组件中的属性，只需要`this.props.[属性名]`来获取使用就可以了。


# redux中间件
redux-logger：能够在控制台清晰的展示出，当前redux操作的流程和信息。（原有状态、派发信息、修改后的状态信息）
redux-thunk：处理异步的dispatch派发
redux-promise：在dispatch派发时，支持promise操作，action传递的参数只能是payload。
在store/index.js中：
```
import {createStore, applyMiddleware} from 'redux';
import reduxLogger from 'redux-logger';
import reduxThunk from 'redux-thunk';
import reduxPromise from 'redux-promise';
let store = createStore(reducer, applyMiddleware(reduxLogger, reduxThunk, reduxPromise));
```
使用redux-promise的注意事项：
action-creator中返回的action对象，传递给reducer的action数据（从服务器获取的数据，开始返回的是一个promise）中的属性名必须是`payload（严格区分大小写）`，只有这样，当Promise成功，中间件才会帮我们重新发送一次派发给reducer，然后把获取的数据更新redux容器中的状态（promise必须放到payload属性下才可以）


<- - - 本文 ღ 结束 - - - >