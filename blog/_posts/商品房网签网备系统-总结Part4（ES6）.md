---
title: 商品房网签网备系统-总结Part4（ES6）
date: 2017-09-30 22:29:55
categories: 项目总结
tags:
- ES6
- 商品房网签网备系统
---
本篇是商品房网签网备系统-总结的第四部分：ES6，主要将项目中所用到的ES6新特性撸一遍，包括let、const、解构赋值、模板字符串、数组的Array.from()和includes()方法、扩展运算符与rest、箭头函数、Set与Map集合、for-of循环、模块等。
<!--more-->

# 块级声明let、const
## let
let用于声明变量，类似于var，不同于var的是：
-  不存在变量提升
一般地，将let声明语句放在块级作用域的顶部。
```
function func() {
    console.log(a); // undefined
    console.log(b); //ReferenceError（变量未声明）
    var a = 10;
    let b = 20;
}
```
- 将变量的作用域限制在当前代码块中
- 不允许重复声明
也就是说，同一作用域内，不能用let重复定义已经存在的标识符，否则就抛SyntaxError（语法错误）
```
var c = 3;
if (true) {
    let c = 5; // 不报错
    console.log(c);
}
console.log(c);
// 5
// 3
```

    >注意：for循环有一个特别之处：循环变量所在的()内部和循环体内部是两个独立的作用域。而函数内部，只有一个作用域。

    ```
    //for循环
    for (let i = 0; i < 2; i++) {
      let i = 'abc';
      console.log(i);
    }
    // abc
    // abc

    //函数
    function func(e) {
      let e; // 报错
    }
    ```

**应用：**
- for循环的计数器，就很合适使用let命令。避免循环变量i泄露为全局变量。
```
for (let i = 0; i < 10; i++) {
  // ...
}
console.log(i); // ReferenceError
```
## const
const用于声明常量，一旦声明不可更改。需要注意的是：
- 必须初始化
- 重复赋值会抛出SyntaxError。（这点不同于let）
- 对于对象和数组，const只能保证指针是固定的，但指针绑定的值是可以修改的。（这点不同于其他语言）
因此，将复杂数据类型声明为常量必须非常小心。
```
const person = { name: "chen" };
//可以修改对象属性的值
person.name = "xinhua";

person = { name: "wang" }; //TypeError（分配给常量变量）
```
**应用：**
多用于声明（数值、字符串、布尔值）简单类型数据。

# 解构赋值
ES6针对对象和数组添加了解构功能，其本质是将数据结构打散，然后匹配赋值。对象是按**属性名**匹配，数组是按**对应位置**匹配。
## 语法
```
//数组
let [a, b, c] = [1, 2, 3]; //a=1,b=2,c=3
let [a, [b], c] = [1, [2, 3], 4]; //a=1,b=2,c=4
let [a, b, c] = [1, 2]; //a=1,b=2,c=undefined

//对象
let { a, b, c: { d: third } } = { a: 1, b: 2, c: { d: 3 } };
//a=1,b=2,third=3
//c: { d: third }冒号后是{}，将解构深入到了下一个层级
```

需要注意的问题：
- 如果匹配失败，返回undefined。
- 解构可以嵌套解构，将解构深入到了下一个层级
- 可以用“=”为变量指定默认值。当对应值===undefined时，默认值生效

## 应用
- 交换两个变量的值
```
let a = 1;
let b = 2;
[a, b] = [b, a];
```
- 通过rest参数实现不定元素的数组
```
let ["red", ...rest] = ["red", "green", "blue"];
console.log(rest); // ["green", "blue"]
```

- 从函数返回多个值
```
// 返回一个数组
function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象
function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```
- 定义函数参数
```
//数组：参数是有次序的
function add([x, y, z]){}
add([1, 2, 3]); 

[[1, 2], [3, 4]].map(([a, b]) => a + b);
// [ 3, 7 ]


//对象：参数是无次序的
function add({x, y, z}) {}
f({z: 3, y: 2, x: 1});

//可以给参数指定默认值
function move({x = 0, y = 0} = {}) {
  //注意{x, y} = { x: 0, y: 0 }写法是为变量x和y指定默认值
}
move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]

```

- 快速提取JSON数据
```
let responseObj = {
  code: 200,
  msg: "OK",
  data: [867, 5309]
};
let { code, msg, data } = responseObj;
console.log(code, msg, data );
// 200, "OK", [867, 5309]
```
    *本项目中提取从后端返回的JSON数据全部用的解构赋值方法，详见query组件，源码请戳☛☛☛
    [https://github.com/huaz224/realEstate/blob/master/src/components/inc/query.vue](https://github.com/huaz224/realEstate/blob/master/src/components/inc/query.vue)
    line 122-144*

# 模板字符串
语法：由反撇号\`\`表示，在反撇号中用占位符${}嵌入变量或JavaScript表达式。

**增加的新特性：**
- 在反撇号中直接换行，简化多行书写
```
//ES5：使用数组或者"+"拼接
let strA = ['a', 'b', 'c'].join('\n');
let strB = "str\n" + "ing";

//ES6
let strB =`str
ing`;
```
- ${}中可以是任意的JavaScript表达式，可以是变量、运算、函数调用，也可以引用对象属性
```
//运算
let x = 1,
    y = 2,
    msg = `${x} + ${y} = ${x + y}` // "1 + 2 = 3";

//引用对象属性
let person = { name: "chen" },
    who = `${person.name}`;
```
- 反撇号需要转义，单、双引号不需要转义；所有空格、缩进都会被保留并输出
```
let str = `h\`ello'"\nw orld\\`;
console.log(str);
//h`ello'"
//w orld\
```

# 数组Array.from()、includes()
## Array.from()方法
**3个参数**
- 第一个：类数组对象和可迭代（iterable）对象
- 第二个：映射函数，可以使用箭头函数（可选）
- 第三个：指定映射函数的this值（可选）

**应用**
用于将**类数组对象**和**可迭代（iterable）对象**转为真正的数组。
- 类数组对象
类数组对象是具有数值型索引和length属性的对象。常见有DOM操作返回的NodeList集合和函数内部的arguments对象。
```
let arrayLike = {
    '0': 'a',
    '1': 'b',
    '2': 'c',
    length: 3
};
// ES6的写法
let arr = Array.from(arrayLike); // ['a', 'b', 'c']

// arguments对象
function trans() {
    return Array.from(arguments);
    // 等同于return [...arguments];
    //扩展运算符（...）也可以将某些数据结构转为数组。
}
console.log(trans(1, 2, 3));//[1, 2, 3]
```
- 可迭代（iterable）对象
包括Array、Set、Map和String等等都是可迭代对象
```
//string类型
Array.from('hello');//等同于[...'hello']
// ['h', 'e', 'l', 'l', 'o']

//Map集合
let map = new Map();
map.set("未审批", 2);
map.set("未通过", 8);
console.log(Array.from(map)) // [["未审批",2],["未通过",8]]
```
    map集合和Array.from(map)在控制台输出的数据结构如图所示。
![](/img/商品房网签网备系统/Arrayfrom.png)

## includes()方法
ES6为字符串添加了includes()方法，ES7将为数组添加该方法，用于判断数组是否包含给定的值，返回true/false。
**2个参数**
- 第一个：要搜索的值
- 第二个：开始搜索的索引位置（可选）
```
[1, 2, 3].includes(2)     // true
[1, 2, 3].includes(4)     // false
```

**与indexOf()的区别**
- indexOf()找到返回index（第一次出现的位置），否则返回-1；includes()则返回true/false。
- indexOf()使用===进行比较，includes()能够识别NaN
```
[1, 2, NaN].includes(NaN) // true
[1, 2, NaN].indexOf(NaN) // -1
```


# 扩展运算符与rest
## 扩展运算符
rest参数的逆运算。可以将一个数组**拆开**为用，分隔的参数序列。替代数组的apply方法。
```
//函数调用
function f(x, y, z) {
  // ...
}
var args = [0, 1, 2];
f(...args);
```
**应用**
- 合并数组，替代concat方法
```
var arr1 = [3, 4, 5]
// ES5
[1, 2].concat(arr1)  
// ES6
[1, 2, ...arr1]
```

- 将可迭代对象（Array、Set、Map和String）和类数组对象（NodeList、arguments）转为真正的数组
```
[...'hello']
// [ "h", "e", "l", "l", "o" ]
```

- Math.max方法，简化求出一个数组最大元素的写法
```
// ES5 的写法
Math.max.apply(null, [14, 3, 77])
// ES6 的写法
Math.max(...[14, 3, 77])

```
- push函数，将一个数组添加到另一个数组的尾。
```
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
// ES5的写法  //push方法只允许向数组末尾推入一项
Array.prototype.push.apply(arr1, arr2);
// ES6 的写法
arr1.push(...arr2);
```
- date的新写法
```
// ES5
new (Date.bind.apply(Date, [null, 2015, 1, 1]))
// ES6
new Date(...[2015, 1, 1]);
```

## rest运算符
功能与扩展运算符相反，把，隔开的参数序列**组合**成一个数组，主要用于**不定参数**。
特性：
- 每个函数最多只能声明一个rest参数，并且只能放在最后
- 函数的length属性不包括rest参数
- rest参数可以替代arguments对象，由于rest参数中的变量是一个数组，因此这个变量可以使用所有数组的特有方法。
```
function add(...values){
  let sum = 0;
  for  (let value of values){//values=[1,2,3];
    sum += value;
  }
  return sum
}
add(1,2,3);//6
```
注意：区别***函数传参解构***
```
// 参数是一组有次序的值：数组
function f([x, y, z]) { ... }
f([1, 2, 3]);
```


# 箭头函数
ES6允许使用=>定义函数。

## 语法
( 参数 ) => { 函数体 }
- 没有参数时，直接写一个没有内容的()
- 只有一个参数时，参数可不加圆括号()
- 只有一条语句时，函数体可不加大括号{}，并且可以省略return关键字
```
let getName = () => "hua";
let func = v => v++;
let add = (num1, num2) => {
    if (num1 > 0) {
        return num1 + num2;
    } else {
        return num2;
    };
}
```
需要注意的是：
- 如果箭头函数向外返回一个对象字面量，需包裹在()中返回
```
let getObj = id => ({ name: "chen", type: "student" });
//等同于
let getObj = function {
    return {
        name: "chen",
        type: "student"
    }
};
```

## 特性
- 没有this绑定，它总是基于词法作用域，由外层函数调用时的this绑定决定。
- 没有[[ Construct ]]方法，不能通过new关键字调用
- 没有原型，不存在prototype属性
- 不存在arguments对象，如有需要可以使用rest参数代替

## 应用
- 简化回调函数
诸如filter()、forEach()、map()、sort()等高阶函数均可以接收一个箭头函数作为参数，来处理数组。
```
//对楼层进行排序
floors.sort((a, b) => b - a);
```
    源码请戳☛☛☛statList组件line 164-179
    households组件line 201-239
    query组件line 56-113
- 修正定义在函数中的this指针指向问题
```  
let that = this;     
this.statData = this.optionsData.filter( function ( item ){
  return (item.developers_name == that.value);
}); 
// 以上代码等同于   
this.statData = this.optionsData.filter( item => item.developers_name === this.value );
```
    *详见statList组件，源码请戳☛☛☛
    [https://github.com/huaz224/realEstate/blob/master/src/components/gov/statList.vue](https://github.com/huaz224/realEstate/blob/master/src/components/gov/statList.vue)
    line 172-177*
    *另外，本项目所有响应服务器返回数据的函数都写为箭头函数*

# Set、Map集合
## Set集合
Set集合是一种**无重复元素**的有序列表。
- 属性
Set.prototype.constructor：构造函数，默认就是Set函数。调用new Set()创建Set集合
Set.prototype.size：返回Set实例的元素数量。
- 方法
add(value)：添加某个值
has(value)：检测Set集合是否存在某个值，返回true/false。
delete(value)：移除某个值
clear()：清除所有元素
```
let set = new Set();
set.add(5);
set.add("5");
set.add("a");
console.log(set.size); //3
console.log(set.has("a")); //true
set.delete("a");
console.log(set.size); //2
set.clear();
console.log(set.size); //0
```
    注意：Set集合同Array数组一样，不会对所包含的元素进行强制类型转换，5和"5"是两个独立的元素。
- 特性
可以传入数组来初始化Set集合
使用扩展运算符...将Set集合转换为数组
```
let set = new Set([1, 2, 5, 4]),
    array = [...set];
console.log(array); //[1,2,5,4]
```

## Map集合
Map集合是一种存放着许多组**键值对**的有序列表
- 属性
Map.prototype.constructor：构造函数，默认就是Map函数。调用new Map()创建Set集合
Map.prototype.size：返回Map实例的键值对数量。
- 方法
set(key, value)：添加某个键名及其对应的值
get(key)：根据指定的键名获取对应的值，返回value
has(key)：检测指定键名在Map集合中是否存在，返回true/false。
delete(key)：移除指定键名及其对应的值
clear()：清除所有键值对。
```
let map = new Map();
map.set(0, "false");
map.set(1, "true");
map.set("1", "string");
console.log(map.size); //3
console.log(map.get("1")); //"string"
console.log(map.has("1")); //true
map.delete(1)
console.log(map.size); //2
console.log(map.has(1)); //false
map.clear();
console.log(map.size); //0
```
    注意：Map集合键名的判断是调用Object.is()方法实现的，1和"1"是两个键名。这点不同于对象，对象总是将属性名强制转换成string类型
- 特性
可以传入数组来初始化Map集合
使用扩展运算符...将Map集合转换为数组
```
let map = new Map([["name", "chen"], ["age", 24]]),
    array = [...map];
console.log(array); //[["name", "chen"], ["age", 24]]
```

## 应用
- **[...new Set(array)]**对数组**自动去重**，返回一个新数组
- **Array.from(map)**或**[...map]**都可以将一个Map集合转换为一个数组
```
getSummaries({ columns, data }) {
  const sums = [];
  let statusArr = data.map(value => value.status),
    tempMap = new Map();
  columns.forEach((column, index) => {
    if (index === 0) {
      sums[index] = '合计';
    } else if (index === 4) {
      console.log(statusArr, 'statusArr');
      for (let item of [...new Set(statusArr)]) {
        tempMap.set(item, (statusArr.filter(val => val == item)).length);
      }
      sums[index] = Array.from(tempMap).join(" ").replace(/,/g, ":");
    } else {
      sums[index] = '/';
    }
  });
  return sums;
}
```
    *详见households组件，源码请戳☛☛☛
    [https://github.com/huaz224/realEstate/blob/master/src/components/gov/households.vue](https://github.com/huaz224/realEstate/blob/master/src/components/gov/households.vue) 
    line 201-239*

# 迭代器iterator
在ES6中，所有的集合对象Array、Set、Map和String都是可迭代对象，这些对象都具有Symbol.iterator属性，通过这个属性指定的方法能够返回一个迭代器。也就是说，ES6默认为这些内建类型提供了默认的迭代器。
```
let arr = [1, 2, 3];
let iterator = arr[Symbol.iterator]();
console.log(typeof arr[Symbol.iterator]); //"function"
console.log(iterator.next()); //"{value:1,done:false}"
console.log(iterator.next()); //"{value:2,done:false}"
console.log(iterator.next()); //"{value:3,done:false}"
console.log(iterator.next()); //"{value:undefined,done:true}"
```
- 任意可迭代对象或类数组对象（NodeList、arguments）都可以通过Array.from或...转换为数组，比如map的**Array.from(map)**或**[...map]**

# for-of循环
ES6引入的for-of循环，可以遍历所有可迭代对象和类数组对象。
## 原理
for-of循环每执行一次都会调用可迭代对象的next()方法，并将迭代器返回的结果对象的value属性存储在一个变量中，循环将持续执行这一过程直到返回对象的done属性的值为true。
## 语法
```
for (let value of iterableObj){
  console.log(value);
}
```
## 特性
### 集合对象的iterator
3种集合对象Array、Set、Map内建了3种迭代器：
- entries() 返回一个迭代器，其值为多个含有键和值两元素数组
- values() 返回一个迭代器，其值为集合的值
- keys() 返回一个迭代器，其值为集合中所有的键名
其中，Array、Set默认迭代器是values()，Map默认迭代器是entries()
```
let colors = ['red', 'blue', 'pink'];
for (let entry of colors.entries()) {
    console.log(entry);
}
//[0 ,'red']
//[1 ,'blue']
//[2 ,'pink']
let set = new Set([3, 2, 5]);
for (let entry of set.entries()) {
    console.log(entry);
}
//[3, 3]
//[2, 2]
//[5, 5]
let map = new Map([
    ["name", "chen"],
    ["age", 24]
]);
for (let entry of map.entries()) {
    console.log(entry);
}
//["name", "chen"]
//["age", 24]
```

### 字符串的的iterator
ES6通过改变字符串的默认迭代器，使得[]语法操作字符而不是编码单元，因此可以访问双字节字符。
```
let msg = "a 哈b";
for (let m of msg) {
  console.log(m);
}
// "a"
// " "
// "哈" //for循环会输出两个" "
// "b"
```
### DOM集合的iterator
 DOM元素集合，比如NodeList对象也拥有默认的迭代器
```
//html
<div id="main">
    <div id="content"></div>
</div>
<div id="footer">
</div>

//script
let divs = document.getElementsByTagName("div");
console.log(divs, "divs"); //[object HTMLCollection]
for (let div of divs) {
    console.log(div.id);
}
//"main"
//"content"
//"footer"
```


## 应用
- for-of与解构一起使用
使用这种方法后，不需要使用Map集合的内建方法取出每一个键和值。
```
let map = new Map([
    [name, "chen"],
    ["age", 24]
]);
for (let [key, value] of map.entries()) {
    console.log(key,value);
}
//"name" "chen"
//"age" 24]
```
- 在数组中使用for-of，可根据数组中的每一项来做一些操作
比如为对象数组中的每一个对象添加属性
```
for (let item of data) {
  //dyh为单元号
  //sjc属性几乎就是楼层号，但有几个数据错误无法使用
  //bdczl为具体地址，比如"兴城市四家屯街道滨核街1-8号楼2单元2-17-4室"
  let temp = `${(item.bdczl.split('-'))[2]}`;
  //data的每个对象新增floor属性存放楼层号
  item.floor = temp.replace(/\d+[A-X]+/g, function(value) {
    switch (value) {
      case "3A":
        return "4";
      case "12A":
        return "13";
      case "12C":
        return "14";
    }
  });
  if (!floors.includes(item.floor)) {
    floors.push(item.floor);
  }
  //data的每个对象新增door属性存放户号
  item.door = `${item.bdczl.charAt(item.bdczl.length-2)}`
  //data的每个对象新增houseNum属性存放完整门牌号
  item.houseNum = `${item.dyh}-${item.floor}-${item.door}`;
}
```
    还可以根据每一项进行筛选，结合push()方法创建一个新数组
    ```
    for (let val of floors) {
      newData.push({
        flo: val,
        doorArr: (data.filter(item => val == item.floor)).sort(compare)
      });
    }
    ```

    *详见query组件,源码请戳☛☛☛
    [https://github.com/huaz224/realEstate/blob/master/src/components/inc/query.vue](https://github.com/huaz224/realEstate/blob/master/src/components/inc/query.vue)
    line 56-113*

# 模块
ES6以前，JS定义的一切都放在全局作用域中，ES6引入模块之后，在模块中创建的变量不会自动被添加到全局作用域中。***模块的真正魔力在于可以私有化自己的作用域，静态地导入导出你需要的变量，而不是“共享一切”。***
模块功能主要由两个命令构成：export和import。export命令用于导出绑定，import命令用于导入其他模块的绑定。
## export命令
可以使用export关键字将任何变量、函数或类声明导出，未显示导出的都是模块私有的。
**两种写法：**
- 导出声明：将export关键字放在每一个声明的前面
```
//example.js
export let name = 'Michael';
export let age = 27;
```
- 导出引用：先声明，在尾部export导出（推荐使用）
```
//example.js
let name = 'Michael';
let age = 27;

export {name, age};
```
    **需要注意的是：**export导出的变量、函数或类声明必须要有一个名称。因此，匿名函数或类必须用default关键字导出。

## import命令
export导出的绑定只有用import关键字引入之后才可使用。
**语法：**
    import {导入的绑定} from 模块文件的路径
```
// main.js
import {name, age} from './example.js';
```
注意：
- 导入一个绑定还是多个都要使用大括号{}
- 模块文件的路径加上文件扩展名表示本地文件，以区分node.js中的包
- import导入的绑定是只读绑定，不能给导入的绑定重新赋值，有点const的味道

## default命令
模块的默认值是通过default关键字指定的单个函数、变量或类，每个模块**只能有一个**默认值。
**区别与默认导入导出:**
- import导入的绑定不适用大括号{}
- 匿名函数或类必须用default关键字导出
```
//method.js
export default function(x, y) {
  return x * y;
}
//引入
import add from "./method.js"//import命令可以为匿名函数指定任意名字。
```

## 特性
- 导入和导出不能出现在if语句或函数内部，不能有条件导出或以任何方式动态导出
- 可以使用as关键字为导入导出的绑定重命名

<- - - 本文 ღ 结束 - - - >