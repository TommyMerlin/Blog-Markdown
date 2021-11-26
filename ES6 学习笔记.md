---
title: ES6 学习笔记
date: 2021-05-19 14:21:53
tags:
- 前端
- ES6
- JavaScript
categories:
- 前端
- 语言基础
---

## 1.新增关键字

**var** 关键字**缺陷**：

- var 可以重复声明。
- var 无法限制修改。
- var 没有块级作用域。

<!-- more -->

ES6 新增 let 和 const 关键字来声明变量，用法类似于 var。

**let** 关键字：

- let 声明的变量，只在 let 命令所在的代码块内有效。

- let 命令不存在变量提升。

- let 命令不允许在相同作用域内，重复声明同一个变量。

**const** 关键字：

- const 命令声明一个只读的常量。一旦声明，常量的值就不能改变。
- const 命令声明的常量不得改变值。即一旦声明，就必须立即初始化。
- const 命令声明的常量，只在声明所在的块级作用域内有效。
- const 命令声明的常量不提升，只能在声明的位置后使用。
- const 命令声明的常量，与 let 一样不可重复声明。
- const 命令声明的复合类型的数据（对象、数组等），变量指向内存地址（对象里的内容可以更改，但是引用的对象不能更改）。

## 2.箭头函数

ES6 允许使用箭头 => 定义函数。

- 不需要参数或需要多个参数，需用圆括号。
- 代码块部分多于一条语句，就用大括号包裹，并使用 return 返回。
- 箭头函数返回**对象**时，须在对象外面加上括号。

实例：

```js
const a1 = () => {console.log("Hello 1");};

const a2 = (x, y) => console.log(x + y);

const fun1 = function a1() {
    console.log("Hello 2");
    console.log(this);
}

let arr = [1, 6, 3, 7, 2]
let narr = arr.sort((a, b) => a - b)

const fun = x => ({ x: x, name: 'ZhangSan' })
```

关于箭头函数中的 **[this 的指向](https://www.bilibili.com/video/BV1Pz4y1S7Uv?p=31)**：

- **普通函数**的 this：指向它的调用者，如果没有调用者则默认指向 window。

- **箭头函数**的 this：指向箭头函数定义时所处的对象，而不是箭头函数使用时所在的对象，默认使用父级的 this。

综上：箭头函数没有自己的 this，它的 this 是继承而来，默认指向在定义它时所处的对象（宿主对象）。

```js
const obj1 = {
    fun1: function fun1() {
        console.log(this)
    }
}
function add() {
            console.log(this)
        }
const obj2 = {
    fun1: () => console.log(this)
}

// {fun1: f}
// 有调用者，指向调用者
obj1.fun1();
// Window {window: Window, self: Window, document: document, name: "", location: Location, …}
// 没有调用者，默认指向 window
add();
// Window {window: Window, self: Window, document: document, name: "", location: Location, …}
obj2.fun1();
```

## 3.数组的新增高级方法

```js
let numsBefore = [2, 5, 20, 10, 50, 30, 7, 20];
let numsAfter = numsBefore
                .filter(x => x > 10)		// 过滤
                .sort((a, b) => a - b)		// 排序
                .map(x => x * 0.5)			// 映射
                .reduce((s, n) => s + n, 0)	// 汇总
console.log(numsAfter)
```

## 4.解构赋值和扩展运算符

解构赋值

```js
let [a, b, c] = [1, 2, 3]
let user = {
    name: "ZhangSan",
    age: 18,
}

let {name, age} = user     // 对象的解构赋值名称属性必须对应
```

扩展运算符

```js
// 展开数组
let arr1 = [1, 2, 3]
let arr2 = [...arr1, 4, 5, 6]
// arr2 = [1, 2, 3, 4, 5, 6]

// 可变参数
function show(...args) {
    args.forEach(x => console.log(x));
}

show(1, 2, 3, 4)
```

## 5.对象的新用法

### 5.1 ES 6 的 Class（类）的概念

```js
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }

    setName(name) {
        this.name = name;
    }
}

const p = new Person("ZhangSan", 20);
```

子类继承父类

```js
class Student extends Person {
    constructor(name, age, school) {
        super(name, age);
        this.school = school;
    }
}

const s = new Student("ZhangSan", 20, "ZJU")
```

### 5.2 JSON 对象的新应用

- JSON.stringify()  串行化
- JSON.parse()       反串行化

```js
let name = "ZhangSan";
let age1 = 12;

const obj = {
    name,        // 键和值名称相同时可简写
    age: age1,
    say(){
        console.log("Hello");
    }
}

let json = JSON.stringify(obj)
```

## 6.Module 模块化编程

模块化优点：

- 减少命名冲突。
- 避免引入时的层层依赖。
- 提升执行效率。

**export 命令**：用于规定模块的对外接口

- 一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。如果希望外部能够读取模块内部的某个变量，就必须使用 export 关键字输出该变量。

**import 命令**：用于输入其他模块提供的功能

- import 命令接受一对大括号，里面指定要从其他模块导入的变量名。大括号内的变量名必须与被导入模块对外接口的名称相同。

## 7.Promise 的原理及应用

主要用于**异步计算**：

- 可以将异步操作队列化，按照期望的顺序执行，返回符合预期的结果。

- 可以在对象之间传递和操作 promise，帮助我们处理队列。

**异步回调**的问题：

之前处理异步是通过纯粹的回调函数的形式进行处理很容易进入到回调地狱中，剥夺了函数 return 的能力。问题可以解决，但是难以读懂，维护困难。稍有不慎就会踏入回调地狱，嵌套层次深，不好维护。

**primise**

promise 是个对象，对象和函数的区別就是对象可以保存状态，函数不可以（闭包除外）并未剥夺函数 return 的能力，因此无需层层传递 callback，进行回调获取数据。代码风格容易理解，便于维护多个异步等待合并便于解决。

```js
new Promise((resolve, reject) => {
    console.log("Level 1")

    let flag = false;

    if (flag) {
        resolve("Level 2")
    }
    else {
        reject("error")
    }

}).then(res => {
    console.log(res)
}).catch(e => {
    console.log(e)
})
```

