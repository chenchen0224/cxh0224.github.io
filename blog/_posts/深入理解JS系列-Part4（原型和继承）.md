---
title: 深入理解JS系列-Part4（原型和继承）
date: 2017-12-08 23:56:29
categories: 读书笔记
tags:
- JavaScript
- 深入理解JS
---
本篇是***深入理解JS系列 ***的Part4，我们将要探讨的是JavaScript区别于传统OO语言的新的继承机制：原型和继承。JavaScript中没有类，继承是通过原型来实现的。ES6虽然引入了class关键字，但只是语法糖，JavaScript仍然是基于原型的。
原型和继承是JavaScript中的一个难点。我们只有深入理解原型和继承，才能构建如谷歌一样的大型复杂应用。

参考书籍：《JavaScript高级程序设计（第3版）》
<!-- more -->

# 理解原型对象
## 当我们定义了一个函数之后
```
function Person(name) {
    this.name = name;
}
```
我们创建了一个Person函数，看看发生了什么
- 自动为该函数创建一个***prototype属性***，这个属性是一个指针，指向该函数的***原型对象 ***。
- 默认的，所有的原型对象都会自动获得一个***constructor（构造函数）属性 ***，这个属性又指回该函数。

用图表示就是：
![](/img/深入理解JS系列/func.jpg)

用代码来表示就是：
```
typeof (Person.prototype) === "object";// true
Person.prototype.constructor === Person;// true

```

>注意：Person函数其实就是一个普通的函数，只是通过new关键字来调用可以创建一个对象。所谓构造函数其实就是***以构造对象的形式 ***来调用的***普通函数 ***。

## 然后，用new来调用函数，创建了一个对象
我们用new来构造调用了Person，创建了一个对象person1
```
function Person(name) {
    this.name = name;
}
var person1 = new Person("hua");
```
实际上会经历以下四个步骤：
1）创建一个新对象
2）将这个新对象绑定到this
3）这个新对象会直接复制一份Person中定义的属性保存到自身
4）返回这个新对象
这里需要注意的是：创建的新对象实例会自动获得一个***隐藏属性[[prototype]]***，这个属性指向Person的原型对象。[[prototype]]是一个***内部属性 ，不可访问，***但多数浏览器都***支持访问\_\_proto\_\_属性***（是两个下划线哦！！！）
用代码表示就是：
```
person1.__proto__; // {constructor:ƒ Person(name)}
person1.__proto__ === Person.prototype; // true
```
用图表示就是：
![](/img/深入理解JS系列/obj.jpg)


## 为原型对象添加方法
- 如果我们在构造函数Person中添加方法
```
function Person(name) {
    this.name = name;
    this.sayName = function() {
        console.log(this.name);
    };
}
var person1 = new Person("hua");
person1.sayName();// hua
var person2 = new Person("san");
person2.sayName();// san
```

以上代码用图表示就是：
![](/img/深入理解JS系列/add1.jpg)
在构造函数中添加方法的主要问题就是：person1和person2中都有一个sayName()的方法，但这两个方法***并不是同一个Function的实例 ***，因为函数也是对象，每创建一个实例，就会实例化一个函数对象。
可以用以下代码来证明：

```
person1.sayName === person2.sayName;// false
```

这样的话，并不能做到函数的复用，况且有this存在，所有的实例完全可以共享一个sayName()方法，重复定义会造成内存的浪费。

- 如果直接在Person的原型对象中添加方法
```
function Person(name) {
    this.name = name;
}
Person.prototype.sayName = function() {
    console.log(this.name);
};
var person1 = new Person("hua");
person1.sayName();// hua
var person2 = new Person("san");
person2.sayName();// san
```
以上代码用图表示就是：
![](/img/深入理解JS系列/add2.jpg)
这里需要注意的是：
- 在Person函数中定义的属性和方法都是***实例属性 ***，对象实例可以***直接复制 ***，获得的是这些实例属性的***一个副本 ***
- 而在原型对象中定义的属性和方法是***所有实例共享的***，对象实例可以***间接引用 ***，person1和person2中的sayName()方法***是同一个Function的实例 ***
可以用以下代码来证明：
```
person1.sayName === person2.sayName;// true
person1.name === person2.name;// false
```
***因此，我们应该在构造函数中定义实例化的属性，在原型对象中定义方法和共享的属性，这样每个实例都会有自己的一份实例属性的副本，但同时又共享着对方法的引用。***



# 原型链
在上面的栗子中，我们展示了实例person1和它的原型对象Person.prototype的关系，我们顺着Person的原型对象向上追溯，顶端到底指向什么呢？？？
我们注意到，Person的原型对象指向Object.prototype。它们之间***通过属性[[prototype]]来连接 ，形成了一条原型链  ***，原型链的顶端为Object.prototype：
person1 - - - > Person.prototype - - - > Object.prototype 
用图表示就是：
![](/img/深入理解JS系列/chain.jpg)

## 基于原型链的搜索机制
当我们需要读取对象的某个属性时，会执行一次搜索操作，搜索首先从对象实例开始，如果在对象实例中没有找到该属性，就会沿着原型链继续向上搜索，找到了就立刻停止，否则会直至顶端。**这就是基于原型链的搜索机制。**
比如，在上面的原型链中搜索person1.sayName()，会经历以下两个步骤：
1）搜索实例person1 
2）搜索Person.prototype，找到了sayName()方法，搜索过程停止
这种搜索机制会产生一种***屏蔽效应 ***：
当为对象实例添加一个属性时，这个属性就会屏蔽原型链上层的***同名属性 ***。屏蔽只是一个阻止访问的操作，并没有修改原型链。

## Object.prototype
我们知道，在ECMAScript中，所有的对象都是Object的实例，因此所有的原型对象默认都会包含一个指针，指向Object.prototype。
所以，这也就是为什么我们没有定义hasOwnProperty()、toString()等方法，而创建的每个对象却都能调用它们的真正原因。
同样，所有的原生引用类型（Array、String等等）之所以可以调用很多种方法也是因为在它们的原型中定义了这些方法。

## 关于constructor属性的澄清
- 对象实例，自身是没有constructor属性的
person1是一个对象实例，自身是没有constructor属性的，之所以可以访问constructor属性，是因为向上搜索在它的原型Person.prototype中找到了constructor属性。
用代码证明如下：
```
person1.constructor === Person;// true
console.log("constructor" in person1);// true
console.log(person1.hasOwnProperty("constructor"));// false
```
- 原型对象被重写之后会丢失constructor属性
Person.prototype的constructor属性是在***Person函数声明时的默认属性 ***，一旦以一个新的对象来重写了Person.prototype，那么这个***新的对象是不会自动获得constructor属性 ***的。


## 重写原型对象
如果以对象字面量的语法来重写整个原型，代码示例如下：
```
function Person(name) {
    this.name = name;
}
Person.prototype = {
    sayName: function() {
        console.log(this.name);
    }
};
var person1 = new Person("hua");
person1.sayName();// hua
```
以上代码用图表示就是：
![](/img/深入理解JS系列/rewrite.jpg)
我们会发现，将一个以对象字面量形式创建的新对象赋给Person.prototype后，你会发现，***这个新的原型对象自身是并不具备constructor属性，但却可以沿着原型链向上搜索它的原型Object.prototype。因此，这个新的原型对象的constructor属性不再指向Person函数了，而是指向Object构造函数。***
用代码证明如下：
```
Person.prototype.constructor === Person;// false
Person.prototype.constructor === Object;// true
```
当constructor的指向很重要的时候，我们可以手动改变原型对象的constructor指向
```
Person.prototype = {
    constructor: Person,
    sayName: function() {
        console.log(this.name);
    }
};
```
改变后，如图所示：
![](/img/深入理解JS系列/constructor.jpg)
我们会发现，改变constructor的指向并没有影响原型链，这货到底有什么用处，目前真没发现。

# 继承
JavaScript中只有对象，没有类！！！它的继承相对于传统OO语言的类继承更为简单：***一个对象可以继承在另一个对象中定义的属性和方法。  ***


## 原型继承
***在上面的栗子中，其实沿着原型链就实现了继承。处于原型链底端的对象可以向上继承。***
但是，如果两个对象并不在一条原型链上，它们之间并无关联。那么，怎么实现这两个对象之间的继承呢？？？
核心思想就是：***将一个对象的原型指定为要继承的这个对象的实例，从而将两个对象关联起来。本质是重写原型对象。***
我们知道，使用构造函数可以new出来一个对象实例，如果将该实例赋给另一个对象的原型，是不是就实现了继承了呢？？？答案是肯定的。
可能不太好理解，我们来看一个栗子：
```
function A() {
    this.A = "i am a";
}
A.prototype.sayA = function() {
    console.log(this.A);
};

function B() {
    this.B = "i am b";
}
B.prototype = new A();// 实现B继承自A
B.prototype.sayB = function() {
    console.log(this.B);
};
var b1 = new B();
b1.sayB();// i am b
b1.sayA();// i am a
```
以上代码定义了两个类型：A和B，通过创建A的实例new A()，并将该实例赋给B的原型对象，实现了B继承自A。实现的本质是以一个类型A的实例重写了B的原型对象。
以上代码用图表示就是：
![](/img/深入理解JS系列/BnewA.jpg)
这里有几点需要注意：
- B的实例b1具有一个实例属性B，可以访问三个原型链中的属性A、sayB、sayA。注意***属性A（黄色）存在于B.prototype中  ***，这是因为属性A是实例属性，而B.prototype是函数A的一个实例。
- ***B的原型对象被重写之后，丢失了constructor属性。***b1.constructor访问的是A.prototype中的constructor属性。
用代码证明如下：
```
b1.constructor.name === A;// true
```
如果constructor属性非常重要，可以手动指回。

这种继承存在的问题：
+ 子类很轻易就可以重写父类原型上的方法
B.prototype.\__proto__.sayA = null;
+ 父类实例的私有属性以及公有属性都变为子类实例的公有属性=>`玩不了私有`
+ `子类B默认的原型`上的属性和方法，会废弃

缺点：父类的构造函数如果有`引用类型的属性`时，继承后会变成子类的原型中的属性，会`被子类的所有实例共享`。


## call继承
原理：A.call(this);把`父类A作为普通函数执行`，让A中的this变为B的实例，相当于**`只是给B的实例增加一些属性和方法`**。
注意：`子类和父类在原型链上并没有建立连接`，它们的原型都指向Object.prototype。
继承的形式：
```
function A() {
    this.A = "A";
}
function B() {
    //this：B的实例
    A.call(this);
    this.B = "B";
}
```
弊端：无法继承父类A原型上的属性和方法，仅仅是通过call执行给子类的实例增加了一些属性和方法。=>`玩不了公有`

缺点：只能继承父类构造函数中的属性，原型对象中的属性和方法无法继承。`无法实现函数的复用`。

## 寄生组合继承
组合式继承：
原理：A的私有变为B的私有，A的公有变为B的公有。
```
function A() {
    this.A = "A";
}
function B() {
    A.call(this);
    this.B = "B";
}
B.prototype = A.prototype;
B.prototype.sayB = function () {}
```
存在的问题：
一般都不这样处理，因为这种模式可以轻易修改父类A原型上的东西（重写“太方便”了），这样会导致A的其它实例也受到影响。

常见的处理方案：
```
B.prototype = Object.create(A.prototype);
```
`Object.create(obj)`：内置Object类的方法，主要用于`基于给定的对象来创建它的一个空的对象实例`。
作用：
- 创建一个空对象
- 让新创建的空对象的\__proto__指向给定的对象（把obj作为新创建空对象的原型）

使用Object.create()
我们知道，原型链连接的是对象和对象，与函数无关。那么我们是否能够绕开函数，而仅仅对两个对象之间建立关联关系呢？？？
ECMAScript5中引入了一个新方法：Object.create()，这个方法的第一个参数是一个用作原型对象的对象，返回一个新对象实例。我们来看这个函数：
```
function create(o) {
    function F() {}
    Foo.prototype = o;
    return new F();
}
```
在create内部，首先定义了一个空函数F，并将其原型指向参数o，o是一个原型对象，最后返回了函数F的一个实例。***本质上，就是对给定的对象执行了一次浅复制。  ***
这个函数可以用于：***基于给定的对象来创建它的一个对象实例。***
我们来看下面这个栗子：
```
var a = {
    A: "i am a"
};
var b = Object.create(a);
console.log(b.A); // i am a
```
在上边这个栗子中，我们将对象a作为一个基础对象，传入到了Object.create()函数中，然后返回了一个对象b，它的原型是对象a。

Object.create()适用于：不需要兴师动众的创建构造函数，而只是想***保持两个对象类似 ***的情况。

使用Object.create()所带来的问题：当给定的对象包含引用类型值的属性时，执行浅复制会只复制一个指针，这会***导致该引用类型值的共享  ***。
我们来看一个栗子：
```
var a = {
    color: ['red', 'blue']
};
var b1 = Object.create(a);
b1.color;// ["red", "blue"]
b1.color.push('pink');

var b2 = Object.create(a);
b2.color;// ["red", "blue", "pink"]
b1.color.push('yellow');

a.color;// ["red", "blue", "pink", "yellow"]
```
我们注意到，a.color不仅属于对象a，而且还被b1和b2共享。修改三个对象中的任意一个对象的color，其他对象也都会改变。


寄生组合式继承的原理：
- 创建一个`没有任何属性的空对象`
- 这个空对象既是子类B的原型，它的\__proto__又指向A的原型
    + 子类通过`A.call(this);`继承了父类的私有属性
    + 子类的实例就可以`基于原型链的查找机制`，调用父类A原型上的属性和方法

好处：避免了A的私有属性变为B的公有属性

## ES6中的继承
ES6创建类是有自己标准语法的。
语法：
- ES6创建的类`只能用new`来调用，不能当做普通函数来执行
- static：把类当做一个普通的对象，设置私有方法（和实例没有关系），注意`只能设置方法不能设置属性`，设置属性只能在类外边基于传统JS设置

```javascript
class Fn {//Fn是类名，没有小括号
    contructor(n,m){
    //等价于ES5类的构造体=>私有属性
        this.x = n;
        this.y = m;
    }
    //=>给Fn的原型上设置方法：只能设置方法，不能设置属性
    getX(){
        
    }
    static AA(){

    }
}
Fn.prototype.BB = 100;
let f = new Fn(10, 20);
//可以基于传统JS给Fn添加属性和方法
// Fn.prototype.getX=function(){}
// Fn.prototype.BB=100;
// Fn.AA=function(){} //=>把Fn当做一个普通对象设置的私有方法(和实例没关系)
```


基于ES6实现继承：
```javascript
class A {
    constructor(m) {
        this.x = m;
    }
    getX() {
        console.log(this.x);
    }
}
class B extends A {//=>extends类似于实现了原型继承
    constructor() {
        super('i am a');//=>类似于call继承：在这里super相当于把A.constructor.call()给执行了，并且让方法中的this是B的实例，super当中传递的实参都是在给A的constructor传递的
        this.y = 200;
    }
    getY() {
        console.log(this.y);
    }
}
let B1 = new B();
```


# 最后想说
哎呀，妈妈呀！我终于写完了！！！这是我最耗费经历的一篇博了，为了彻底理清原型和继承，我参考了很多的资料，画了很多的图，反复的看资料，反复的修改，真的不容易，好在收货了很多。学习芝士的心情贼好！！！
最后，奉上参考的资料：
参考书籍：
- 《JavaScript高级程序设计（第3版）》第6章
- 《你不知道的JavaScript（上卷）》第二部分 第5章

参考文档：
- [MDN：对象原型](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Objects/Object_prototypes)
- [MDN：继承与原型链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)

<- - - 本文 ღ 结束 - - - >