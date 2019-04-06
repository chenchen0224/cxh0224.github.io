---
title: 深入理解JS系列-Part6（AJAX）
date: 2018-02-08 11:52:29
categories: 读书笔记
tags:
- JavaScript
- 深入理解JS
---
本篇是***深入理解JS系列 ***的Part6：AJAX。在项目中，如果我们要和后端进行数据交互，那就要学习AJAX。

<!-- more -->
## 什么是AJAX
`Asynchronous JavaScript and XML`
- Asynchronous
在AJAX中的异步不是我们理解的同步异步编程，而是泛指`局部刷新`，但是在以后的AJAX请求中尽可能使用异步获取数据（因为异步数据获取不会阻塞下面代码的执行）
- XML
XML是一种文件格式（可以把HTML理解为XML的一种）：可扩展的标记语言。
作用：可自己扩展一些语义标签，来存储数据和内容。=>清晰的展示数据结构。


## 如何创建Ajax实例
抛开第三方的工具和框架，原生JS实现Ajax。
主要考察以下几点：
XMLHttpRequest对象的工作流程
- 第一步：创建一个对象
IE5、IE6用的是ActiveXObject对象

- 第二步：启动一个请求以备发送
open([Http Method], [URL], [Async] , [user-name], [user-pass])
规定HTTP协议的请求类型，URL地址，是否异步。
	+ Http Method：请求方式
	+ URL向服务器发送请求的API（Application Programming Interface） 接口地址
	+ Async设置请求的同步异步
- 第三步：事件监听：一般监听是readystatechange事件（`AJAX状态改变`事件），基于这个事件可以获取服务器返回的响应头、响应主体内容
	+ AJAX状态
`0`：unsent =>刚开始创建xhr，还没有发送
`1`：opened =>已经执行了open这个操作
`2`：headers_received =>已经发送AJAX请求（AJAX任务开始），响应头信息已经被客户端接收了（响应头中包含了：服务器的时间、返回的HTTP状态码...）
`3`：loading =>响应主体内容正在返回
`4`：done =>响应主体内容已经被客户端全部接收，可以在浏览器端使用了。
```javascript
xhr.onreadystatechange = ()=>{
	if(xhr.readyState==4 && xhr.status==304){
		//=>xhr.responseText;
	}
}
```

- 第四步：发送请求。
从这步开始，`当前AJAX任务开始`，如果AJAX是同步的，后续代码不会执行，要等到AJAX状态成功后再执行，反之异步不会。
xhr.send([请求主体内容]);
	+ 没有请求主体，为null；
	+ 有请求主体，一般是字符串，可能是JSON格式、XML格式、普通字符串格式的。
=>真实项目中常用的是`URL-encode格式`的字符串（"id=1000&lx=2000"）
响应
responseText获得字符串形式的响应数据。

栗子：
```javascript
let xhr=new XMLHttpRequest();//=>0
xhr.open('GET','/temp/list',true);//=>1
xhr.onreadystatechange=()=>{//=>DOM事件绑定是异步的
   if(xhr.readyState===2){console.log(1);}
   if(xhr.readyState===4){console.log(2);}
};
xhr.send();//=>当前AJAX任务开始，异步
console.log(3);
//=>3 1 2
```

```javascript
let xhr=new XMLHttpRequest();
xhr.open('GET','/temp/list',true);
xhr.send();
xhr.onreadystatechange=()=>{
   if(xhr.readyState===2){console.log(1);}
   if(xhr.readyState===4){console.log(2);}
};
console.log(3);
//=>3 1 2
```

```javascript
let xhr=new XMLHttpRequest();
xhr.open('GET','/temp/list',false);
xhr.onreadystatechange=()=>{
   if(xhr.readyState===2){console.log(1);}//=>监听到了状态改变为2，但主任务队列AJAX任务没有完成，被占着，没有执行
   if(xhr.readyState===4){console.log(2);}
};
xhr.send();//=>任务开始（同步：只要当前AJAX任务没有完成，就什么也不做不了）
console.log(3);
//=>2 3
```
当AJAX任务开始，由于是同步编程，主任务队列在状态没有变成4（任务结束）之前一直被这件事占用着，其它事情都做不了
==AJAX同步的弊端：==
1）阻塞下边同步代码的执行
2）阻碍自己状态的监听。只有在状态变为4时，才触发readystatechange事件绑定的方法。
（当服务器把响应头返回的时候，状态为2，触发了事件readystatechange，但是由于主任务队列没有完成，被占着呢，绑定的方法也无法执行... 所有只有状态为4的时候执行一次这个方法）
==AJAX任务执行的一个特殊性：==
当AJAX任务是同步任务时，AJAX完成会`优先执行自己的readystatechange事件（虽然这个事件在等待任务队列）`，然后才执行后边的同步代码。例如，上边的栗子输出的是2 3，而不是3 2。

```javascript
let xhr=new XMLHttpRequest();
xhr.open('GET','/temp/list',false);
xhr.send();
xhr.onreadystatechange=()=>{
   if(xhr.readyState===2){console.log(1);}
   if(xhr.readyState===4){console.log(2);}
};
console.log(3);
//=>3
```



## xhr的属性和方法
[属性]
- xhr.response：响应主体内容
- xhr.responseText：响应主体的内容是`字符串`（JSON或XML格式字符串）
- xhr.responseXML：响应主体的内容是XML文档

- xhr.status：返回的HTTP状态码
- xhr.statusText：状态码的描述

- xhr.timeout：设置请求超时的时间
- xhr.withCredentials：是否允许跨域（false）

```javascript
//open
//xhr.timeout = 1000;//=>请求超时会自动中断
//xhr.ontimeout = () => {
//	console.log('请求超时');
//}
xhr.onabort = () => {
	console.log('请求被强制中断');
}
//onreadystatechange
setTimeout(()=>{
	xhr.abort();
},1000);
```

[方法]
- xhr.open()：打开URL请求
- xhr.send()：发送AJAX请求
- xhr.abort()：强制中断AJAX请求（当前建立连接的HTTP连接通道到干掉了）
- xhr.setRequestHeader([key], [value])：设置请求头
（不能出现中文，必须在`open之后`才可以设置成功）
编码：`encodeURIComponent`
解码：`decodeURIComponent`
- xhr.getAllResponseHeaders()：获取所有的响应头信息
- xhr.getResponseHeader([key]) ：获取key对应的响应头信息
- xhr.overrideMimeType()：重写MIME类型


 

```javascript
    xhr.onreadystatechange = () => {
        if (!/^(2|3)\d{2}$/.test(xhr.status)) return; //=>证明服务器已经返回内容了（HTTP请求成功）
        if (xhr.readyState === 2) {
            //=>响应头信息已经回来了
            let time = xhr.getResponseHeader('date');
            console.log(time, new Date(time));
        }
        if (xhr.readyState === 4) {
            console.log(xhr.responseText);
        }
    };
```
- `xhr.getResponseHeader('date')` ：获取响应头中的服务器时间。
`AJAX状态为2`时获取，获取的是格林尼治时间（`字符串`，中国标准时间比它多8小时）。
- new Date()：获取当前客户端时间（`对象`）
new Data(时间字符串)：把指定的时间字符串格式化为标准的北京时间（Data类的实例）
时间字符串的格式：常用的有，"xxxx-xx-xx xx:xx:xx"或"xxxx/xx/xx"  



从服务器端获取时间会产生一个问题：服务器返回数据需要时间，所以客户端拿到返回的服务器时间和真实的时间是有误差的。
//=>那么如何减小时间误差？
1）在AJAX为2，就从响应头中获取时间，而不是等到状态变为4  =>请求方式设置为head，只获取响应头，不需要响应主体内容
2.特殊：即使向服务器发送一个不存在的请求地址，返回的是404状态码，但是响应头信息中都会存在服务器时间（不建议使用，不友好）


## JQ中的AJAX
1. 两种写法：\$.ajax([URL],[options]) 或 $.ajax([options])
在options中有一个url字段代表请求的URL地址
`$.get` /`$.post` / `$.getJSON` / `$.getScript` 这些方法都是基于\$.ajax构建出来的快捷方法，项目中最常使用的还是\$.ajax

2. options的配置项：
- `url`：请求的API接口地址
- `method`：请求的方式     
- `data`：传递给服务器的数据
GET请求是基于问号传参传递过去的，POST请求是基于请求主体传递过去的 。
data的值可以是对象也可以是字符串：中文会自动编码
		+ 对象类型（用的最多）：JQ会把`对象转换`为 xxx=xxx&xxx=xxx 的模式（`application/x-www-form-urlencoded`）
		+ 查询字符串：写什么就传递什么
     
- `dataType`：预设置获取结果的数据格式
支持的格式：json、jsonp、xml、html、script、text
（服务器返回给客户端的响应主体中的内容一般都是字符串[JSON格式居多]）
注意：设置返回结果为json格式 ，`并不会影响服务器返回的结果`，只是JQ内部对返回的结果进行了`二次处理`，最终转为JSON格式的对象给我们。 
     
- `async`：设置同步或者异步（默认true异步）
- `cache`：设置get请求下是否建立缓存，（默认true->建立缓存，false不建立缓存）。
当设置为false时，并且设置当前请求是get请求，JQ会在请求的URL地址末尾追加随机数（`时间戳`：`+( new Date() )`）。

- `success`：回调函数，当AJAX请求成功执行的回调函数，会把响应主体中的内容（可能二次处理了）传递给回调函数。
回调函数的参数：
           result：服务器获取的结果
           textStatus：状态描述
           jqXHR：JQ封装的XHR，和原生的不太一样
      
- `error`：请求失败后执行的回调函数

3. 局限性
无法解决回调地狱的问题。如果发多次AJAX请求，只能在success的回调中再次发请求，这样嵌套，可维护性很差。用的很少，现在大部分项目都是基于Promise对AJAX进行管控。

## axios的AJAX
它是一个类库，是一个基于Promise管理的AJAX库。
axios只是一个`普通的函数`，在这个`函数上`（而不是它的原型上）定义了很多方法，准确的来说，axios并不是一个类。

1. 提供了对应请求方式的方法（例如：get/post/head/delete/put/options...）
- `axios.get([url], [config]);`
get请求中，会把`params中的键值对`拼接成URLEncode格式字符串，然后以问号传参的方式，传递给服务器，类似于JQ-AJAX中的data
```
    axios.get('url', {
        params: {
            name: 'chen',
            age: 9
        }
    });
```

- `axios.post([url], [data], [config]);`
post请求中，配置项传递的内容都相当于基于请求主体传递给服务器，但是传递给服务器的内容格式是`RAW`（`JSON格式的字符串`），不是x-www-form-urlencode
```
    axios.post('url', {
        name: 'chen',
        age: 9
    });
```


2. 基于get或post发请求，返回的结果都是`Promise实例`
基于axios发送请求，第一个then接收到的参数是一个`对象`，包含以下属性：
	+ `config`：基于axios发送请求做的配置项
	+ `data`：从服务器获取的响应主体内容
	{code: 0, data: {…}}
	+ `headers`：从服务器获取的`响应头信息`
	content-type:"application/json; charset=utf-8"
	+ `request`：创建的AJAX实例
	+ status：状态码
	+ statusText：状态码的描述

```javascript
	axios.get('url', {
        params: {
            lx: 12
        }
    }).then(result => {
        //console.log(result);//=>获取的结果是一个对象
        let {data} = result;
    }).catch(msg => {
        console.log(msg);//=>请求失败的原因
    });
```
基于axios解决回调地狱：
先请求A，A完成做什么，然后请求B，B完成做什么。
```javascript
    axios.get('urlA', {
        params: {
            lx: 12
        }
    }).then(result => {
        let {data} = result;
        //...
        return axios.post('urlB');
    }).then(result => {
        let {data} = result;//=>result是B成功后的结果
        console.log(data);
    });
```



3. 一次并发多个请求：`axios.all([ary])`
适用于多个请求都完成后做什么。
执行then方法，接收到的参数是一个`数组`，分别存放每一个请求返回的结果。
```javascript
    let sendAry = [
        axios.get('urlA'),
        axios.get('urlB'),
        axios.post('urlC')
    ];
    //=>三个请求都完成才做一些事情（可以基于ALL实现）
    axios.all(sendAry).then(result => {
        console.log(result);//=>是一个数组，分别存储每一个请求的结果
        let [resA, resB, resC] = result;
        //axios.spread((resA, resB, resC) => {}
    });
```
`axios.spread()`：请求都完成触发这个函数
原理是JS中的柯理化函数思想
```javascript
    module.exports = function spread(callback) {
        return function wrap(arr) {
            return callback.apply(null, arr);
        };
    };
    axios.all(sendAry).then(axios.spread((resA, resB, resC) => {
        //=>RES-A/-B/-C分别代表三次请求的结果
    }));
```

4. 初始化常用的一些配置项
全局配置：
- 基础URL
`后台服务器URL的统一前缀统一提取`
```
axios.defaults.baseURL = 'https://www.easy-mock.com/mock/5b0412beda8a195fb0978627/temp';
```
- 设置拦截器
设置响应拦截器：分别在响应成功和失败的时候做一些拦截处理（在响应成功后设定的方法之前，会先执行拦截器中的方法）
```javascript
    axios.interceptors.response.use(function success(result) {
	    //=>成功之前拦截
        return result.data;//=>传给下边then方法的result
    }, function error() {
	    //=>失败之前拦截
    });
    //=>使用
    axios.get('/list', {
        params: {
            lx: 12
        }
    }).then(result => {//=>拦截这个方法的执行
        console.log(result);
    });
```    
- `transformRequest`选项允许我们在请求发送到服务器之前对请求的数据做出一些改动
只适用于以下请求方式：`put/post/patch`
```javascript
	//=>将请求主体内容格式化为xxx-www-urlencode格式
    axios.defaults.transformRequest = data => {
        //=>DATA:就是请求主体中需要传递给服务器的内容（对象）
        let str = ``;
        for (let attr in data) {
            if (data.hasOwnProperty(key)) {
                str += `${key}=${data[key]}&`;
            }
        }
        return str.substring(0, str.length - 1);
    };
    //qs.stringify(data)
```
qs插件的两个方法：
`qs.parse()`
`qs.stringify()`
- 定义哪些状态码是成功
```
    axios.defaults.validateStatus = function validateStatus(status) {
        //=>自定义成功失败规则：RESOLVE / REJECT（默认规则：状态码以2开头算作成功）
        return /^(2|3)\d{2}$/.test(status);
    };
```
- 定义`请求头`
注意：结果result中的headers是响应头的信息。
- 定义请求主体
- 定义请求超时时间 

```
    //=>一般不定义为公共的
    axios.defaults.timeout = 3000;
    axios.defaults.headers={
        name:'chen'
    };
    axios.defaults.params={};//=>GET传参
    axios.defaults.data={};//=>POST传参
``` 

使用： 一般全局配置常用基础URL、拦截器、请求主体、状态码这几个。
注意：在请求中的配置要高于全局默认配置的。   

```
    //=>使用
    axios.get('/list', {
        params: {
            lx: 12
        },
        headers: {xxx: 'xxx'}//=>作为请求头传递进去
    }).then(result => {
        console.log(result);
        //=>result.headers：服务器返回的响应头信息
    });

    //=>POST：三个参数 axios.post(url[,data][,config])
    axios.post('/add', {
        lx: 12,
        sex: 1
    }, {
        headers: {xxx: 'xxx'}//=>作为请求头传递进去
    }).then(result => {
        console.log(result);
    });
```



<- - - 本文 ღ 结束 - - - >