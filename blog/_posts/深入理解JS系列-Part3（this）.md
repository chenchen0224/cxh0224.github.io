---
title: 深入理解JS系列-Part3（this）
date: 2017-11-12 21:52:29
categories: 读书笔记
tags:
- JavaScript
- 深入理解JS
---
本篇是***深入理解JS系列***的Part3：this。this是一个非常有魔力的对象，它被自动定义在函数内部。this到底指向谁？在函数调用时它被绑定到了哪个对象？我们今天将要对这些问题进行讨论。

参考书籍：《JavaScript高级程序设计（第3版）》《你不知道的JavaScript》
<!-- more -->
# ES6之前
在ES6之前，this在任何情况下都不指向它的词法作用域。也就是说，***this机制只关注函数在哪里调用，它指向谁是在函数运行时确定的。  ***

接下来，我们根据函数的调用分四种情况来解析this。
## 作为普通函数被调用
如果一个函数作为普通函数被调用，也就是不带任何修饰参数进行调用，那么***this就默认指向window***。
我们来看一个栗子。
```
var a = "global";
function foo(){
	console.log(this.a);
}
foo();// "global"
```

## 作为对象的方法来调用
### this指向当前对象
函数作为对象的一个属性，并且***以对象的属性形式被调用时，this就指向当前对象***。
```
var a = "global";
var obj = {
    a: 2,
    foo: function() {
        console.log(this.a);
    }
};
obj.foo(); // 2
```
在上边的这个栗子中，函数foo()被调用时前面加上了对obj的引用，将this绑定到了obj。
这里需要注意的是：
无论foo函数是在obj中声明，还是在全局作用域中声明然后再添加为obj的引用属性，这个函数严格来说都不属于obj。个人理解，obj只是用foo保存了一个对这个函数的引用。

### 例外情况
我们知道，函数名不加()只是一个指向该函数的指针。***如果将一个函数指针以值的形式进行传递  ***，会发生什么呢？？？***this会丢失绑定的对象 ，默认指向window***.
我们来看一个栗子。
```
var a = "global";
var obj = {
    a: 2,
    foo: function() {
        console.log(this.a);
    }
};
var bar = obj.foo; //函数别名
bar();// "global"
```
以上代码将obj.foo赋给bar，bar保存的是一个对foo函数本身的引用，此时，调用bar()函数其实就是在全局作用域中调用的foo()函数，因此，this指向window。

看到这，你是不是觉得this很简单？那你就大错特错了，this这么有魔力的对象，其实是很复杂的，，，
栗子再进阶。Let's go !!!
当obj.foo作为一个参数被传入到了另一个函数中执行，也就是作为回调函数被传递，看看会发生什么，，，
```
var a = "global";
var obj = {
    a: 2,
    foo: function() {
        console.log(this.a);
    }
};
function test(fn) {
	var a = 3;
    fn();
}
test(obj.foo); // "global"
```
在上边的栗子中，将obj.foo作为一个参数传入test()函数，我们知道，函数传参是按值进行传递的，其实就是将foo函数的一个引用传递给了fn。fn是在test()函数中被调用的，但是谁调用了它呢？？？并没有一个确定的对象调用fn，因此foo中的this将绑定到全局对象window。
如果将obj.foo传入JavaScript内置的函数呢？？？
```
var a = "global";
var obj = {
    a: 2,
    foo: function() {
        console.log(this.a);
    }
};
setTimeout(obj.foo, 100);// "global"
```
上面的setTimeout()函数其实类似于下面的伪代码：
```
function setTimeout(fn, delay){
	// 等待delay毫秒
	fn();
}
```
本栗中的setTimeout()函数的内部运行机制其实和上面的test()函数是一样的。如果在全局作用域中没有找到变量a，就会返回undefined。
***回调函数丢失this绑定是非常常见的。*** 解决这个问题最常用的办法就是使用self = this 机制，或者使用ES6的箭头函数，这两部分内容稍后会讲哦。。。
```
var a = "global";
var obj = {
    a: 2,
    foo: function() {
        var self = this;
        setTimeout(function timer() {
            console.log(self.a);
        }, 100);
    }
};
obj.foo(); // 2
```

## 作为构造函数被调用
如果一个函数作为构造函数被调用，***this就代表它new出来的对象***。但构造函数作为普通函数调用例外。
```
var a = "global";
function Foo(a){
	this.a = a;
}
var bar = new Foo(2);
bar.a;// 2
```
以上代码用new关键字调用了普通函数Foo()，创建了一个对象实例bar，函数Foo()内部的this会被绑定到bar。
此部分的内容会在本系列的Part3（原型和继承）中详细讲解，这里就不再啰嗦了。

## 使用call()、apply()、bind()来调用
JavaScript提供的绝大多数函数以及自己定义的所有函数都可以使用call()、apply()、bind()方法来调用，这三个方法都可以***显式的指定this的绑定对象 ***，我们称之为***显式绑定 ***。
- call()、apply()
这两个方法的第一个参数都会传入一个对象，是给this准备的，当函数在调用时就会将这个对象绑定到this。主要区别在于第二个参数，apply()传入数组或arguments对象，call()必须将其余参数逐个列出。
```
var a = "global";
var obj = {
    a: 2
};
function foo(){
	console.log(this.a);
}
foo.call(window);// "global"
foo.call(obj);// 2
```
当运行foo.call(obj)时，foo函数体内的this被强制指向了obj，于是结果是2。

- bind()
bind()方法的第一个参数，用于设置this的值，会返回一个新函数。
```
var a = "global";
var obj = {
    a: 2
};
function foo(){
	console.log(this.a);
}
var func = foo.bind(obj);
func();// 2
```
在这里，foo()函数调用了bind()并传入对象obj，创建了一个函数func，func内部的this值等于obj。

- 还有一种效果等同于显示绑定的情形
数组许多内置的高阶函数，比如filter()、forEach()、map()等等，提供了一个可选的参数来指定this的值。
举例来说：
```
var obj = {
    a: "i am a"
};
function foo(el) {
    console.log(el, this.a);
}
[1, 2].forEach(foo, obj);

// 1 "i am a"
// 2 "i am a"
```
本栗调用foo()时，就把this值绑定到了obj。如果在forEach()函数中传入箭头函数，就会忽略obj，因为箭头函数会在词法上绑定this，这个稍后会讲哦。。。


# self = this 机制
self = this 机制可以很好的解决this绑定的问题，其实它***使用的是一个我们非常熟悉的工具：词法作用域   ***。它不仅可以修正闭包中的this，还可以修正回调函数中的this。
还记得红宝书中那个在闭包中使用this的的栗子么，我们来看一下
```
var a = "global";
var obj = {
    a: 2,
    foo: function() {
        return function() {
            console.log(this.a);
        }
    }
};
var bar = obj.foo();
bar(); // "global"
```
以上代码在调用obj.foo()时会返回一个匿名函数，并将这个匿名函数赋值给bar，bar就是一个闭包，保存了foo函数的作用域以及对外部作用域的引用。当调用bar()时，结果返回"global"，为什么this没有指向obj呢？？？
这是因为***this只存在与创建它的那个函数内部，永远不能访问外部函数中的this。***那么，我们是不是可以***将外部作用域中的this保存在一个闭包能够访问到的变量里   ***。
如下代码：
```
var a = "global";
var obj = {
    a: 2,
    foo: function() {
    	var self = this;
        return function() {
            console.log(self.a);
        }
    }
};
var bar = obj.foo();
bar(); // 2
```
将外部作用域中的this保存在self变量里，基于词法作用域查找self就可以修正匿名函数中的this绑定。


# ES6引入了箭头函数
ES6引入了箭头函数，将this与它的词法作用域关联起来。具体来说，就是***箭头函数中的this会“继承”外层函数调用的this绑定  ***。其实等同于ES6之前的self = this 机制。 
箭头函数常常用于简化回调函数。
还记得商品房网签网备系统的那个例子么
```  
this.statData = this.optionsData.filter( item => item.developers_name === this.value ); 
// 以上代码等同于 
let self = this;     
this.statData = this.optionsData.filter( function ( item ){
  return (item.developers_name == self.value);
});  
```

*详见statList组件，源码请戳☛☛☛
[https://github.com/huaz224/realEstate/blob/master/src/components/gov/statList.vue](https://github.com/huaz224/realEstate/blob/master/src/components/gov/statList.vue)
line 172-177*

# 被忽略的this
当显示绑定call()、apply()、bind()方法的第一个参数传入null或undefined时，函数内部的this会被忽略。
那么什么情况下会传入null呢？？？
## 使用apply()把数组展开成参数
我们来看一个栗子。
```
function foo(a, b) {
    console.log(a, b);
}
foo.apply(null, [1, 2]);// 1 2
```
ES6引入了扩展运算符...，可以代替apply()来展开数组。以上代码等同于：
```
foo.apply(...[1, 2]);// 1 2
```

## 使用bind()对参数进行柯里化
我们知道，使用bind()会返回一个新的函数，那么如何让这个新的函数携带上层函数传过来的参数呢？就是将bind()的第一个参数设置为null，其余参数传给这个新的函数，这种技术我们称之为柯里化。
```
function foo(a, b) {
    console.log(a, b);
}
var func = foo.bind(null, 3)
func(4);// 3 4
```
以上代码，foo调用bind()返回一个func函数，foo作为func的上层函数，将参数3传给了func函数，对应第一个参数a = 3。

<- - - 本文 ღ 结束 - - - >
