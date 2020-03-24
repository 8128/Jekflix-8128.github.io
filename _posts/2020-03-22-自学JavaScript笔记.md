---
layout: post
title:  自学入门JavaScript笔记
date:   2020-03-22 20:20:00
description: 前端入门之路
subtitle: 
comments: true
image: https://raw.githubusercontent.com/8128/PicGo/master/20200322203138.png
optimized_image: 
category: tutorial
tags:
  - tutorial
  - code
  - javascript
author: tty
paginate: false
---

因为工作原因，我需要学习Node.JS。作为一个在JavaScript上毫无建树的人，我只能从头学起。这是我的学习笔记。

本笔记基于udemy课程JavaScript basics for beginners，你可以在YouTube上看到相关试听视频

## Prerequisite

1. 安装live server 插件 in vscode

2. 在项目新建一个hello world的html，右键你的html然后用live server打开

3. 假如你html有改动，在html中的内容会自动刷新到live server

基础：

- js脚本可以添加到head或添加到body，建议添加到body的末尾

- chrome中alt+command+i打开console

## Variables

在ES6之前，js常常用var来定义变量，但在ES6之后var不再被推荐，建议使用let来定义变量

在JavaScript中假如你没有define一个变量则其值为undefined

在JavaScript中一般用单引号来括起string

Variable不能是保留字（reserved keyword），最好有意义，并且不以数字开头，不能有空格和短杠-（hyphen），命名方式同样遵循camel notation（驼峰命名法），variable都是case-sensitive的

可以同时initialize多个变量，用逗号隔开

### constants

假如你使用的是const而不是var，那么你的变量不能被修改

## Primitive types

JavaScript共有两种value可以assign给variable，一种是primitives(aka. value types)，一种是reference type，

### rimitives

- tring
- Number（在JavaScript中没有float，double这种类型，只有统一的number）
- Boolean
- undefined（是一种type，同时是一个value）
- null

假如当前我们有一个变量，我们没有它的值，但是知晓这个variable将来是什么type，这时候我们就将它设为null，而不是设为undefined

### Dynamic typing

编程语言有两种，一种是static，一种是dynamic。在static language里，变量设置了type之后就不能再更改。但是在JavaScript中，变量能被修改为其他数据类型。

## Reference types

### Objects

```javascript
let name = 'Mosh';
let age = 30;
let person = {
  name = 'Mosh', //value pair, key is called properties
  age = 30
}; // this is an object literal

// Two way to access the properties
// 1. Dot notation
person.name = 'John';

// 2. Bracket notation
person['name'] = 'Mary';
```

Dot notation is more concise and shorter, and it should be your default notation

But sometimes the variable you wanna use might be dynamic, like

```javascript
let selection = 'name';
person[selection] = 'Mary'
```

In this case the second notation would be better

### Arrays

```javascript
let  selectedColors = ['red','blue']; // these square brackets are called array literal
selectedColors[2] = 1; // we can store different types of data into the array
```

Type of the array is object

## Functions

```javascript
function greet(parameter) {
	console.log('Hello ') + name;
}

greet(Your argument);
```

## Operators

I will ignore the basic common stuff

### Arithmetic Operators

- Addition + (increment ++)
- Subtraction - (decrement --)
- Multiplication *
- Division /
- Remainder of division %
- exponentiation **

### Assignment Operators

- +=
- -=
- =

### Comparasion Operators

- \>=
- <=
- ==
- ===(strict equality operator, have the same type and the same value)
- !==
- ==(loose equality operator)

=== 判断规则:

1. 如果类型不同，就[不相等]
2. 如果两个都是数值，并且是同一个值，那么[相等]；(！例外)的是，如果其中至少一个是NaN，那么[不相等]。（判断一个值是否是NaN，只能用isNaN()来判断）
3. 如果两个都是字符串，每个位置的字符都一样，那么[相等]；否则[不相等]。
4. 如果两个值都是true，或者都是false，那么[相等]。
5. 如果两个值都引用同一个对象或函数，那么[相等]；否则[不相等]。
6. 如果两个值都是null，或者都是undefined，那么[相等]。

== 判断规则：

1. 如果两个值类型相同，进行 === 比较。
2. 如果两个值类型不同，他们可能相等。根据下面规则进行类型转换再比较：
3. 如果一个是null、一个是undefined，那么[相等]。
4. 如果一个是字符串，一个是数值，把字符串转换成数值再进行比较。
5. 如果任一值是 true，把它转换成 1 再比较；如果任一值是 false，把它转换成 0 再比较。
6. 如果一个是对象，另一个是数值或字符串，把对象转换成基础类型的值再比较。对象转换成基础类型，利用它的toString或者valueOf方法。js核心内置类，会尝试valueOf先于toString；例外的是Date，Date利用的是toString转换。非js核心的对象，令说（比较麻烦，我也不大懂）
7. 任何其他组合，都[不相等]。

### Ternary Operators

: ? 三元运算符

### Logic Operators

- : ?

- &&
- ||

the result of a logic operators is not always a boolean value. Eg, false || 'Mosh' === 'Mosh'

#### Falsy

1. undefined
2. null
3. 0
4. false
5. ''
6. NaN

Anything else is truthy

it will return the first truthy value in the ||, eg, false || 1 || 2, it returns 1

### Bitwise Operators

- `|` bitwise or
- `&` bitwise and



## Control flow 

I will ignore the parts which is the same as Java (if...else..., while, do...while, switch)

### for loop

```javascript
const person = {
	name: 'Mosh';
  age: 30
};

for (let key in person) 
	console.log(key, person[key]);

const colors = ['red','green','blue'];
//get you the index with 'in'
for (let index in colors) 
  console.log(index, colors[index]);

//get you the data inside of the array with 'of'
for (let color of colors)
  console.log(color);

// in 和 of的区别就在于，in是在iterate key，而of在iterate value
```

## Object

```javascript
// an example of object
const circle = {
  radius: 1,
  loaction: {
  	x: 1;
  	y: 1
	},
  isVisible: true,
  draw: function() {
    console.log('draw');
  }
};

circle.draw();
```

### Factory Function

```javascript
function createCircle(radius) {
  return {
    radius,
    draw() {
      console.log('draw');
    }
  };
}
```

using this way to create object, the object.constructor will shows you Object()

### Constructor Function

```javascript
function Circle(radius) {
	this.radius = radius;
  this.draw = function() {
    console.log('draw');
  }
}
```

This one looks like a class, but the factory function looks like a function

### Dynamic Nature

```javascript
const circle = {
  radius: 1
};

circle.color = 'yello'; // add properties
circle.draw = function() {}

delete circle.color; // remove properties
delete circle.draw;
```

### Cloning an object

```javascript
const another = Object.assign(newObj, oldObj);

const another = {...target}; // three dots means all the properties inside of the old object
```

### Garbage Collection

All the process is automatic and you cannot clean the memory mannualy.

### String

There is primitive string and object String, primitive string can also use functions, and they will be wrapped as String object

Escape character(转义字符)

在JavaScript中，引号间的换行会被记录

```javascript
const message = 'hello';
const another =
‘This is my ${message}
'first' message';
```

输出会是

```
This is my hello
'first' message
```

## Array

### adding element

及时你使用了const来作为array，你虽然不能reassign this array to something else，但是你可以增加删除其中的元素。

```javascript
const numbers = [3,4];
//add at the end
numbers.push(5,6);
//add at the beginning
numbers.unshift(1,2);
//middle
//第一个：位置，第二：删除的元素数量，第三以及之后：添加的内容
numbers.splice(2,0,'a','b');

```

### Find element

```javascript
// it is easy to find a primitive type
numbers.includes(1);
numbers.indexOf(1);
numbers.lastIndexOf(1);

// to find an object
const course = courses.find(function(course) {
  return course.name === 'xyz';
});
```

### Arrow functions

looks like java lambda

```javascript
const course = courses.find(course => course.name === 'a');
```

### Removing Element

```javascript
const numbers = [1,2,3,4];

//end
numbers.pop();

//beginning
numbers.shift();

//middle
numbers.splice(2, 3); //first is offset, second is the number of elements you want to delete
```

### Empty an array

```javascript
//Solution 1
let num = [1,2,3];
let num1 = num;

num = []; //in this solution, num is reassigned but num1 is still pointing to the num, so the info of num will not be garbage collected

//Solution 2
numbers.length = 0;

//Solution 3
numbers.splice(0, numbers.length);

//Solution 4
while (numbers.length > 0) {
  numbers.pop();
}
```

### Combing Arrays

```javascript
const first = [1,2,3];
const second = [4,5,6];
const combined = first.concat(second);
const sllice = combined.slice(2,4);
```

### Iterate Array

```javascript
for (let num of nums) {
	console.log(num);
}
nums.forEach(function(number){
  console.log(number);
});
// after es6
nums.forEach(number => console.log(number));
nums.forEach((number, index) => console.log(index, number)); // the second value is index, the first is the value
```

### Join array

```javascript
const numbers = [1,2,3]
const joined = numbers.join('-');//default is comma
// result is 1-2-3
```

### Sort

```javascript
const numbers = [1,2,3]
numbers.sort();
numbers.reverse();
```

### Every and some

```javascript
const numbers = [1,2,3]
const k = numbers.every(function(value){
  return value >= 0;
});// all passed then return true, else return false

const k = numbers.some(function(value){
  return value >= 0;
});// at leaset one match then return true, else return false
```

### Filter an array

```javascript
const filtered = numbers.filter(function(value){
  return value >= 0;
});

const filtered = numbers.filter(value => value >= 0);
```

### Mapping

```javascript
const items = filtered.map(n => ({value : n}));
```

### Reduce

```javascript
array.reduce(function(total, currentValue, currentIndex, arr), initialValue)

const sum = numbers.reduce((accumulator, currentValue) => {
  return accumulator + currentValue;
});
```

## Function