---
layout: post
title:  JS异步讲解-Asynchronous JavaScript
date:   2020-03-25 17:15:00
description: promise到底是什么？await和async怎么用？
subtitle:
comments: true
image: https://raw.githubusercontent.com/8128/PicGo/master/20200322203138.png
optimized_image:
category: javascript
tags:
  - tutorial
  - code
  - javascript
author: tty
paginate: false
---

头都被JavaScript的异步处理搞炸了，而且脑子不太好使，经常学一点忘一点。做个笔记记录一下几个学习JavaScript过程中的痛点

部分笔记来源：udemy-modern JavaScript from the beginning

[cnblog](https://www.cnblogs.com/moltboy/archive/2013/04/24/3040213.html)

## 什么是异步 Asynchronous

要说清楚异步，需要先把同步说明清楚。所谓同步就是一步步向下走

```javascript
posts = loadPostsSync();
doTheNextThing();
```

如上所示，必须先把第一个函数执行完了，第二个函数才能执行。众所周知，JavaScript不支持多线程，那么node.js又是怎么实现高并发的呢？就是靠异步。现在的环境下一个工程往往会受制于读写时间，而不是真正的处理运算时间。假如我们遇到了读写瓶颈，我们就暂时先搁置手上的工作，优先去处理下一步的内容，随后等到读写完成，再次回来捡起地上的工作，这就实现了异步。

```javascript
loadPostsAsync(function (){
	//wait til posts are fetched
	//do something with posts
});

doTheNextThing();
```

## 回调函数

首先要把callback说清楚，callback就我理解，就是在一个函数中call另外一个函数，避免在异步过程中出错。从简单的setTimeOut到匿名函数，都算callback。在JavaScript中，函数属于对象，是用Function()构造函数创建的Function对象。Function对象包含一个字符串，字符串包含函数的javascript代码。对于javascript来说，这很平常。数据和代码之间的区别是很模糊的。

```javascript
//可以这样创建函数
var fn = new Function("arg1", "arg2", "return arg1 * arg2;");
fn(2, 3);   //6
```

### 传递函数作为回调

```javascript
function fn(arg1, arg2, callback){
    var num = Math.ceil(Math.random() * (arg1 - arg2) + arg2);
    callback(num);　　//传递结果
}

fn(10, 20, function(num){
   console.log("Callback called! Num: " + num);
});　　　　//结果为10和20之间的随机数
```

这里看到我们把function拿来当参数直接传入函数中，这其实就是回调

传统函数以参数形式输入数据，并且使用返回语句返回值。理论上，在函数结尾处有一个return返回语句，结构上就是：一个输入点和一个输出点。这比较容易理解，函数本质上就是输入和输出之间实现过程的映射。

但是，当函数的实现过程非常漫长，你是选择等待函数完成处理，还是使用回调函数进行异步处理呢？这种情况下，使用回调函数变得至关重要，例如：AJAX请求。若是使用回调函数进行处理，代码就可以继续进行其他任务，而无需空等。实际开发中，经常在JavaScript中使用异步调用，甚至在这里强烈推荐使用！

下面有个更加全面的使用AJAX加载XML文件的示例，并且使用了call()函数，在请求对象（requested object）上下文中调用回调函数。

```javascript
function fn(url, callback){
    var httpRequest;　　　　//创建XHR
    httpRequest = window.XMLHttpRequest ? new XMLHttpRequest() :　　　//针对IE进行功能性检测
　　　　window.ActiveXObject ? new ActiveXObject("Microsoft.XMLHTTP") : undefined;

    httpRequest.onreadystatechange = function(){
      if(httpRequest.readystate === 4 && httpRequest.status === 200){　　//状态判断
          callback.call(httpRequest.responseXML);  
       }
    };
    httpRequest.open("GET", url);
    httpRequest.send();
}

fn("text.xml", function(){　　　　//调用函数
   console.log(this); 　　//此语句后输出
});

console.log("this will run before the above callback.");　　//此语句先输出
```

我们请求异步处理，意味着我们开始请求时，就告诉它们完成之时调用我们的函数。在实际情况中，onreadystatechange事件处理程序还得考虑请求失败的情况，这里我们是假设xml文件存在并且能被浏览器成功加载。这个例子中，异步函数分配给了onreadystatechange事件，因此不会立刻执行。

最终，第二个console.log语句先执行，因为回调函数直到请求完成才执行。

上述例子不太易于理解，那看看下面的示例：

```javascript
function foo(){
    var a = 10;
    return function(){
        a *= 2;
        return a;       
    };   
}
var f = foo();
f(); //return 20.
f(); //return 40.
```

函数在外部调用，依然可以访问变量a。这都是因为javascript中的作用域是词法性的。**函数式运行在定义它们的作用域中（上述例子中的foo内部的作用域），而不是运行此函数的作用域中。**只要f被定义在foo中，它就可以访问foo中定义的所有的变量，即便是foo的执行已经结束。因为它的作用域会被保存下来，但也只有返回的那个函数才可以访问这个保存下来的作用域。返回一个内嵌匿名函数是创建闭包最常用的手段。

### 经典面试题

```javascript
for (var i = 0; i < 5; i++) {
    setTimeout(function() {
        console.log('i: ',i);
    }, 1000);
}
```

这道题的答案是五个五。那么为什么是五个五呢？

1、for循环和循环体外部的console是同步的，所以先执行for循环，再执行外部的console.log。（同步优先）

2、for循环里面有一个setTimeout回调，他是垫底的存在，只能最后执行。（回调垫底）

那么，为什么我们最先输出的是5呢？

非常好理解，for循环先执行，但是不会给setTimeout传参（回调垫底），等for循环执行完，就会给setTimeout传参，而外部的console打印出5是因为for循环执行完成了。

这里涉及到JavaScript执行栈和消息队列的概念，概念的详细解释可以看阮老师的 [JavaScript 运行机制详解:再谈Event Loop - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)，或者看 [并发模型与Event Loop](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)

我拿这个例子做一下讲解，JavaScript单线程如何处理回调呢？JavaScript同步的代码是在堆栈中顺序执行的，而setTimeout回调会先放到消息队列，for循环每执行一次，就会放一个setTimeout到消息队列排队等候，当同步的代码执行完了，再去调用消息队列的回调方法。

在这个经典例子中，也就是说，先执行for循环，按顺序放了5个setTimeout回调到消息队列，然后for循环结束，下面还有一个同步的console，执行完console之后，堆栈中已经没有同步的代码了，就去消息队列找，发现找到了5个setTimeout，注意setTimeout是有顺序的。

那么，setTimeout既然在最后才执行，那么他输出的i又是什么呢？答案就是5。。有人说不是废话吗？

现在告诉大家为什么setTimeout全都是5，JavaScript在把setTimeout放到消息队列的过程中，循环的i是不会及时保存进去的，相当于你写了一个异步的方法，但是ajax的结果还没返回，只能等到返回之后才能传参到异步函数中。

在这里也是一样，for循环结束之后，因为i是用var定义的，所以var是全局变量（这里没有函数，如果有就是函数内部的变量），这个时候的i是5，从外部的console输出结果就可以知道。那么当执行setTimeout的时候，由于全局变量的i已经是5了，所以传入setTimeout中的每个参数都是5。很多人都会以为setTimeout里面的i是for循环过程中的i，这种理解是不对的。

**注意，换作let的话那么输出结果就不一样了，let的结果会是0 1 2 3 4**

ES5中的变量作用域是函数，而let语法的作用域是当前块，在这里就是for循环体。在这里，let本质上就是形成了一个闭包。

## Promises

被叫做promise是因为当他们在处理async的任务时他们可以promise去做一些async结束后的事。一个简单的案例。其中resolve表示接下去要做的事，而reject表示出现error等问题后如何handle

```javascript
const posts = [
  {title: 'Post One', body:'This is post one'},
  {title: 'Post Two', body: 'This is post two'}
];

function createPost(post) {
  return new Promise(function(resolve, reject){
    setTimeout(function() {
      posts.push(post);
      const error = false;
      if(!error) {
        resolve();
      } else {
        reject('Error: Something went wrong');
      }
    }, 2000);
  });
}

function getPosts() {
  setTimeout(function() {
    let output = '';
    posts.forEach(function(post){
      output += `<li>${post.title}</li>`;
    });
    document.body.innerHTML = output;
  }, 1000);
}

createPost({title: 'Post Three', body: 'This is post three'})
.then(getPosts)
.catch(function(err) {
  console.log(err);
});
```

所以整体流程是这样的，一开始先召唤了createPost，随后createPost用setTimeOut将任务拖延了两秒，再将新的post放入了旧的post array中。因为没有error，所以可以接下来召唤resolve

而resolve是then中传入的getPost函数，所以在getPost中我们可以获得三条Post数据

所以，我们的所谓promise就是说我不在函数内部直接定义callback，而是先return一个Promise对象，当我的对象return出来之后，你可以用then来实现我函数里的resolve

如果你的函数中确实出现了error，那么你定义的reject可以靠外部的catch来捕捉

## Promisify

以下内容转自[掘金](https://juejin.im/post/5bc76ff56fb9a05cee1e14a9)

`util.promisify`是在`node.js 8.x`版本中新增的一个工具，用于将老式的`Error first callback`转换为`Promise`对象，让老项目改造变得更为轻松。

在官方推出这个工具之前，民间已经有很多类似的工具了，比如[es6-promisify](https://www.npmjs.com/package/es6-promisify)、[thenify](https://www.npmjs.com/package/thenify)、[bluebird.promisify](http://bluebirdjs.com/docs/api/promise.promisify.html)。

以及很多其他优秀的工具，都是实现了这样的功能，帮助我们在处理老项目的时候，不必费神将各种代码使用`Promise`再重新实现一遍。

### 工具实现的大致思路

首先要解释一下这种工具大致的实现思路，因为在`Node`中异步回调有一个约定：`Error first`，也就是说回调函数中的第一个参数一定要是`Error`对象，其余参数才是正确时的数据。

知道了这样的规律以后，工具就很好实现了，在匹配到第一个参数有值的情况下，触发`reject`，其余情况触发`resolve`，一个简单的示例代码：

```javascript
function util (func) {
  return (...arg) => new Promise((resolve, reject) => {
    func(...arg, (err, arg) => {
      if (err) reject(err)
      else resolve(arg)
    })
  })
}
```

1. 调用工具函数返回一个匿名函数，匿名函数接收原函数的参数。
2. 匿名函数被调用后根据这些参数来调用真实的函数，同时拼接一个用来处理结果的`callback`。
3. 检测到`err`有值，触发`reject`，其他情况触发`resolve`

**resolve 只能传入一个参数，所以`callback`中没有必要使用`...arg`获取所有的返回值**

### 常规的使用方式

> 拿一个官方文档中的示例

```javascript
const { promisify } = require('util')
const fs = require('fs')

const statAsync = promisify(fs.stat)

statAsync('.').then(stats => {
  // 拿到了正确的数据
}, err => {
  // 出现了异常
})
```

以及因为是`Promise`，我们可以使用`await`来进一步简化代码：

```javascript
const { promisify } = require('util')
const fs = require('fs')

const statAsync = promisify(fs.stat)

// 假设在 async 函数中
try {
  const stats = await statAsync('.')
  // 拿到正确结果
} catch (e) {
  // 出现异常
}
```

用法与其他工具并没有太大的区别，我们可以很轻易的将回调转换为`Promise`，然后应用于新的项目中。

### 自定义的 Promise 化

有那么一些场景，是不能够直接使用`promisify`来进行转换的，有大概这么两种情况：

1. 没有遵循`Error first callback`约定的回调函数
2. 返回多个参数的回调函数

首先是第一个，如果没有遵循我们的约定，很可能导致`reject`的误判，得不到正确的反馈。
而第二项呢，则是因为`Promise.resolve`只能接收一个参数，多余的参数会被忽略。

所以为了实现正确的结果，我们可能需要手动实现对应的`Promise`函数，但是自己实现了以后并不能够确保使用方不会针对你的函数调用`promisify`。

所以，`util.promisify`还提供了一个`Symbol`类型的`key`，`util.promisify.custom`。

`Symbol`类型的大家应该都有了解，是一个唯一的值，这里是`util.prosimify`用来指定自定义的`Promise`化的结果的，使用方式如下：

```
const { promisify } = require('util')
// 比如我们有一个对象，提供了一个返回多个参数的回调版本的函数
const obj = {
  getData (callback) {
    callback(null, 'Niko', 18) // 返回两个参数，姓名和年龄
  }
}

// 这时使用promisify肯定是不行的
// 因为Promise.resolve只接收一个参数，所以我们只会得到 Niko

promisify(obj.getData)().then(console.log) // Niko

// 所以我们需要使用 promisify.custom 来自定义处理方式

obj.getData[promisify.custom] = async () => ({ name: 'Niko', age: 18 })

// 当然了，这是一个曲线救国的方式，无论如何 Promise 不会返回多个参数过来的
promisify(obj.getData)().then(console.log) // { name: 'Niko', age: 18 }
复制代码
```

*关于`Promise`为什么不能`resolve`多个值，我有一个大胆的想法，一个没有经过考证，强行解释的理由：如果能`resolve`多个值，你让`async`函数怎么`return`（当个乐子看这句话就好，不要当真）*
*不过应该确实跟`return`有关，因为`Promise`是可以链式调用的，每个`Promise`中执行`then`以后都会将其返回值作为一个新的`Promise`对象`resolve`的值，在`JavaScript`中并没有办法`return`多个参数，所以即便第一个`Promise`可以返回多个参数，只要经过`return`的处理就会丢失*

在使用上就是很简单的针对可能会被调用`promisify`的函数上添加`promisify.custom`对应的处理即可。
当后续代码调用`promisify`时就会进行判断：

1. 如果目标函数存在

   ```
   promisify.custom
   ```

   属性，则会判断其类型：

   1. 如果不是一个可执行的函数，抛出异常
   2. 如果是可执行的函数，则直接返回其对应的函数

2. 如果目标函数不存在对应的属性，按照`Error first callback`的约定生成对应的处理函数然后返回

添加了这个`custom`属性以后，就不用再担心使用方针对你的函数调用`promisify`了。
而且可以验证，赋值给`custom`的函数与`promisify`返回的函数地址是一处：

```
obj.getData[promisify.custom] = async () => ({ name: 'Niko', age: 18 })

// 上边的赋值为 async 函数也可以改为普通函数，只要保证这个普通函数会返回 Promise 实例即可
// 这两种方式与上边的 async 都是完全相等的

obj.getData[promisify.custom] = () => Promise.resolve({ name: 'Niko', age: 18 })
obj.getData[promisify.custom] = () => new Promise(resolve({ name: 'Niko', age: 18 }))

console.log(obj.getData[promisify.custom] === promisify(obj.getData)) // true
复制代码
```

#### 一些内置的 custom 处理

在一些内置包中，也能够找到`promisify.custom`的踪迹，比如说最常用的`child_process.exec`就内置了`promisify.custom`的处理：

```
const { exec } = require('child_process')
const { promisify } = require('util')

console.log(typeof exec[promisify.custom]) // function
复制代码
```

因为就像前边示例中所提到的曲线救国的方案，官方的做法也是将函数签名中的参数名作为`key`，将其所有参数存放到一个`Object`对象中进行返回，比如`child_process.exec`的返回值抛开`error`以外会包含两个，`stdout`和`stderr`，一个是命令执行后的正确输出，一个是命令执行后的错误输出：

```
promisify(exec)('ls').then(console.log)
// -> { stdout: 'XXX', stderr: '' }
复制代码
```

或者我们故意输入一些错误的命令，当然了，这个只能在`catch`模块下才能够捕捉到，一般命令正常执行`stderr`都会是一个空字符串：

```
promisify(exec)('lss').then(console.log, console.error)
// -> { ..., stdout: '', stderr: 'lss: command not found' }
复制代码
```

包括像`setTimeout`、`setImmediate`也都实现了对应的`promisify.custom`。
之前为了实现`sleep`的操作，还手动使用`Promise`封装了`setTimeout`：

```
const sleep = promisify(setTimeout)

console.log(new Date())

await sleep(1000)

console.log(new Date())
复制代码
```

### 内置的 promisify 转换后函数

如果你的`Node`版本使用`10.x`以上的，还可以从很多内置的模块中找到类似`.promises`的子模块，这里边包含了该模块中常用的回调函数的`Promise`版本（都是`async`函数），无需再手动进行`promisify`转换了。

而且我本人觉得这是一个很好的指引方向，因为之前的工具实现，有的选择直接覆盖原有函数，有的则是在原有函数名后边增加`Async`进行区分，官方的这种在模块中单独引入一个子模块，在里边实现`Promise`版本的函数，其实这个在使用上是很方便的，就拿`fs`模块进行举例：

```
// 之前引入一些 fs 相关的 API 是这样做的
const { readFile, stat } = require('fs')

// 而现在可以很简单的改为
const { readFile, stat } = require('fs').promises
// 或者
const { promises: { readFile, stat } } = require('fs')
复制代码
```

后边要做的就是将调用`promisify`相关的代码删掉即可，对于其他使用`API`的代码来讲，这个改动是无感知的。
所以如果你的`node`版本够高的话，可以在使用内置模块之前先去翻看文档，有没有对应的`promises`支持，如果有实现的话，就可以直接使用。

### promisify 的一些注意事项

1. 一定要符合`Error first callback`的约定
2. 不能返回多个参数
3. 注意进行转换的函数是否包含`this`的引用

前两个问题，使用前边提到的`promisify.custom`都可以解决掉。
但是第三项可能会在某些情况下被我们所忽视，这并不是`promisify`独有的问题，就一个很简单的例子：

```
const obj = {
  name: 'Niko',
  getName () {
    return this.name
  }
}

obj.getName() // Niko

const func = obj.getName

func() // undefined
复制代码
```

类似的，如果我们在进行`Promise`转换的时候，也是类似这样的操作，那么可能会导致生成后的函数`this`指向出现问题。
修复这样的问题有两种途径：

1. 使用箭头函数，也是推荐的做法
2. 在调用`promisify`之前使用`bind`绑定对应的`this`

不过这样的问题也是建立在`promisify`转换后的函数被赋值给其他变量的情况下会发生。
如果是类似这样的代码，那么完全不必担心`this`指向的问题：

```
const obj = {
  name: 'Niko',
  getName (callback) {
    callback(null, this.name)
  }
}

// 这样的操作是不需要担心 this 指向问题的
obj.XXX = promisify(obj.getName)

// 如果赋值给了其他变量，那么这里就需要注意 this 的指向了
const func = promisify(obj.getName) // 错误的 this
复制代码
```

### 小结

个人认为`Promise`作为当代`javaScript`异步编程中最核心的一部分，了解如何将老旧代码转换为`Promise`是一件很有意思的事儿。
而我去了解官方的这个工具，原因是在搜索`Redis`相关的`Promise`版本时看到了这个[readme](https://github.com/mjackson/then-redis)：

> This package is no longer maintained. node_redis now includes support for promises in core, so this is no longer needed.

然后跳到了`node_redis`里边的实现方案，里边提到了`util.promisify`，遂抓过来研究了一下，感觉还挺有意思，总结了下分享给大家。

## Async & Await

这是ES7中的内容

所谓async就是直接return一个promise。直接在函数之前加async就能让这个函数返回promise

```javascript
async function myFunc() {
	const promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve('Hello'), 1000);
  });
  const res = await promise;
  return res;
}

myFunc()
	.then(res => console.log(res));
```

因为使用了await，所以res会在promise实现resolve之后才生成

如你所见，promise不定义resolve也是可以用then的。（上述代码中then内的函数没有被指代到任一resolve函数中去）