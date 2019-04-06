---
title: 深入理解JS系列-Part7（跨域）
date: 2018-03-02 23:44:29
categories: 读书笔记
tags:
- JavaScript
- 深入理解JS
---
本篇是***深入理解JS系列 ***的Part7：跨域。什么情况下会发生跨域，以及如何来解决跨域的问题，是我们今天要探讨的内容。

<!-- more -->

# 同源策略及限制
源：包含协议、域名、端口三部分内容。
三个必须完全一样才同源，只要有一个不一样，就跨域了。

同源策略，限制只能操作、访问处于同一个源中的文档或脚本。（就是不支持跨域）
主要限制在以下几个方面：
- Cookie、LocalStorage等
- 获取DOM元素
- AJAX请求


Ajax只适合同源通信，不支持跨域。
WebSocket不受同源策略的限制。
CORS支持同源通信，也支持跨域。
# 什么是跨域
`当前HTML页面的地址`和在`页面中ajax请求的API地址`做比较：
- 如果两个地址的`协议、域名、端口号`都相同，相当于HTML页面从同一个源下根据某个地址获取数据，属于`同源策略请求`，基于ajax是可以直接请求到数据的。
- 如果三者有一个不一样，那就是`非同源策略请求`（跨域请求），使用ajax不能直接获取数据了。
AJAX不能直接发跨域请求。
```
//同源
当前HTML页面地址：http://localhost:8000/A.html
ajax请求的接口地址：http://localhost:8000/queryInfo
//跨域
WB打开HTML页面地址：http://localhost:63342/cros/static/A.html?_ijt=8lgn3b4j3g03alsnk666o8unj7
ajax请求的接口地址：http://localhost:8000/queryInfo
```
真实的项目一般都是前后端分离的，大部分公司都会把后台程序用一个新的服务管理，把客户端程序也用一个新的服务管理，两个服务不是同一个源：这样导致客户端是向其他源发送ajax请求，跨域成为请求的阻碍问题

同源：把客户端程序和服务器程序在一个服务中发布

# 跨域通信的几种方式
## jsonp

1. jsonp的原理：
利用script标签不存在跨域限制实现的，需要客户端和服务器端双方面都支持才可以完成。
在客户端ajax不允许跨域请求，但是很多标签都可以直接的跨域，例如：`script、link、img、iframe等`（这些标签的`src或者href`可以设置任何一个资源请求地址，哪怕是其它源下的，`都不存在跨域限制`，直接可以把内容获取到[除非服务器做特殊处理了]。
=>针对这个特点，真实项目中某些JS文件加载的都是CDN地址）。


2. 实现
- 客户端
1）准备一个全局函数（`传递给服务器的函数必须是全局函数`）
2）创建一个script标签，把需要请求的地址放到src属性上，通过问号传参的方式（一般传递函数的属性名是callback，可以和服务器协商修改），把全局函数传递给服务器。
```
<script>
    function fn(result) {
        console.log(result);
    }
</script>
<script src="http://localhost:9000/queryInfo?callback=fn"></script>
```
遇到script标签，基于src地址立即向服务器发送一个HTTP请求

- 服务器端
1）接收客户端的请求信息（script的src请求都是`GET`方式）
2）获取问号传递参数的内容，也就是callback后面传递的函数名
3）把函数名和需要返回给客户端的数据拼成：`函数名（数据）`格式的`字符串`，并且返回给客户端。例如，'fn({"code":0,"msg":"my name is server"})'
```
app.get('/queryInfo', (req, res) => {
    let fn = req.query.callback,//=>获取客户端传递的函数名
        data = {
            code: 0,
            msg: 'my name is server'
        };
    res.send(`${fn}(${JSON.stringify(data)})`);//=>返回指定格式的内容：函数名(数据)
});
```
- 客户端开始渲染返回的内容
=> 就是`把之前传递进去的函数执行`（也就是把fn执行），把括号中的数据当做参数传递给这个函数fn

这里需要注意的是：
- jsonp必须有服务器端的支持
- 局限性：jsonp`只支持GET请求`，对于POST请求等其它请求方式无法实现，所以真实项目中，只把从服务器获取数据的需求，采用jsonp来处理。

JQ中跨域：将dataType设置为jsonp，即可实现跨域请求。
```
    $.ajax({
        url: 'http://localhost:9000/queryInfo',
        method: 'GET',
        dataType: 'jsonp',
        jsonp:'func',//=>将传递的callback修改为func=
        jsonpCallback: 'fn',//=>不走JQ默认生成的函数名，指定我们自己指定的方法
        success: function (result) {
            console.log(result);//=>JQ会帮我们生成一个全局的随机函数，并且把函数传递给服务器：callback=jQuery111308604155376427673_1530948200354
        }
    })
```

## cors
cors（跨域资源共享）：主要是`服务器`来配置允许跨域的相关`头部信息`。
可以理解为支持跨域通信的Ajax，变种的Ajax。
浏览器在识别用Ajax发送了一个跨域请求时，会在HTTP头部中加一个Origin头部，来允许跨域通信。如果不加这个头部，普通的Ajax遇到跨域通信，浏览器会拦截。

主要是服务器设置：（配置允许跨域的相关头部信息）
1. 允许哪些源（通配符或单独的某个源）向这个服务器发送ajax请求
- `*`允许所有的源访问
- （一旦允许携带凭证过来，则设置*会报错，此时只能设置单一的具体的源）
//=>不使用通配符，是为了保证接口和数据的安全，不能让所有的源都可访问
```
//=>res.header("Access-Control-Allow-Origin", "*");
res.header("Access-Control-Allow-Origin", "http://localhost:8000");
```
2. 是否允许跨域时携带凭证（例如，cookie就是一种凭证，true为允许，设置为false，客户端和服务器之间不会传递cookie，这样session存储就失效了）
```
res.header("Access-Control-Allow-Credentials", true);
```
客户端和服务端的Credentials这个属性都要设置为true，否则客户端不能基于请求头传递给服务端。而且传递凭证给服务器，服务器的允许源设置也不能使用*。
```
    axios.defaults.withCredentials = true;//=>xhr.withCredentials=true
    //axios在某些特定场景下，在发送真实请求之前都会发送一个预请求（OPTIONS）格式的，来验证是否允许跨域，返回成功，才会发送真实的请求
```

3. 允许的请求头部
```
res.header("Access-Control-Allow-Headers", "Content-Type,Content-Length,Authorization, Accept,X-Requested-With,,Cookie");
```
4. 允许的请求方式（options一定要有）
```
res.header("Access-Control-Allow-Methods","PUT,POST,GET,DELETE,HEAD,OPTIONS");
```
5. 设置OPTIONS请求目的：作为一个试探性请求
当客户端需要向服务器发送请求之前，首先发送一个OPTIONS请求，服务器接收到是OPTIONS请求后，看一下是否允许跨域，允许返回成功。
如果服务器不允许跨域，则客户端会出现跨域请求不允许的错误；
=>如果客户端检测到不允许跨域，则后续的请求都不在进行!  
=>客户端AXIOS框架就是这样处理的
```
       if (req.method === 'OPTIONS') {
              res.send('OK!');
              return;
          }
          next();
      });
```



弊端：只能指定一个允许源（不能用通配符和指定多个源），所以目前真实项目中基于CORS实现跨域资源共享是主流方案



## webpack代理（webpack proxy）
webScoket
就是把当前请求的地址，代理到和后台目标服务器一样的地址。
1. 安装webpack-dev-server
2. 配置代理：
React，是在person.json里直接配置。可以一个一个接口配置，也可以直接放一个代理地址。
```
    proxy: {
      '/api': {//=>客户端自己请求的地址
        target: 'http://localhost:8000',//=>代理的地址
        changeOrigin: true,
        secure: false,
        pathRewrite: {//=>地址重写
           '^/api' : ''
        }
      },
      '/getInfo':{
        target: 'http://localhost:8000',
        changeOrigin: true
      }
    }
```
在creat-react-app脚手架中，我们只需要在package.json中设置proxy代理属性，属性值是目标服务器的地址。要求客户端请求的是同源下的地址，如果同源下没有这个接口地址，然后会通过proxy代理到目标服务器的地址。
原理：
webpack-dev-server里的proxy，利用websocket通信先让客户端请求websocket里启动的服务，websocket会把目标服务器对应接口地址的数据拉回来，返回给客户端。所以直接请求同源下没有的接口，也可以获取数据!
=> websocket起的是中间代理的作用。


ngnix反向代理
node作为中间件代理

## 基于iframe实现跨域
iframe可以实现父页面嵌入子页面（父页面中可以基于js获取子页面中的内容）
iframe可以配合以下页面实现跨域：
1. window.name
name是window天生自带的属性，而且有一个特点，同源下，在某些页面中设置name的值，页面关掉或者刷新，上次设置的值不消失，能够一直存储最后一次修改的值信息

2. document.domain
只能处理主域相同，但是子域不同的情况
v.qq.com
s.qq.com

修改本地host：目的是除了localhost域名，其它的域名也可以加端口号访问

3. pastMessage（H5）
H5新增的postMessage()处理跨域的方法。
![](https://upload-images.jianshu.io/upload_images/8059334-e4cf166e023452a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




## hash
URL地址中#号后边的就是hash，hash变动页面并不会刷新，这是用hash跨域的原理。
search（?号后边的查询字符串）改变会刷新页面，因此search不能用于跨域。
A向B发送一条消息：
![](https://upload-images.jianshu.io/upload_images/8059334-9959af7dd3ea7045.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<- - - 本文 ღ 结束 - - - >