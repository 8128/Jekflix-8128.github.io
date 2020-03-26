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

[cnblog](https://www.cnblogs.com/moltboy/archive/2013/04/24/3040213.html) [dayTimeAffect](https://juejin.im/post/5d89844ae51d4561ff6668f3)

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

值得一提的是，你new一个promise后，这个promise会执行其中的所有内容直到出现回调函数（或到定义尾部）为止，随后进入pending状态

### 状态

`pending`: 初始状态, 非 `fulfilled` 或 `rejected`.

`fulfilled`: 成功的操作.

`rejected`: 失败的操作.

### 基本用法

```javascript
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

resolve函数的作用是，将Promise对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；reject函数的作用是，将Promise对象的状态从“未完成”变为“失败”（即从 pending 变为 rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。 Promise实例生成以后，可以用then方法分别指定resolved状态和rejected状态的回调函数。

```javascript
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

### then()

它的作用是为 Promise 实例添加状态改变时的回调函数。前面说过，then方法的第一个参数是resolved状态的回调函数，第二个参数（可选）是rejected状态的回调函数。

```javascript
getJSON("/post/1.json").then(function(post) {
  return getJSON(post.commentURL);
}).then(function funcA(comments) {
  console.log("resolved: ", comments);
}, function funcB(err){
  console.log("rejected: ", err);
});
```

then方法返回的是一个新的Promise实例（注意，不是原来那个Promise实例）。因此可以采用链式写法，即then方法后面再调用另一个then方法。

### catch()

Promise.prototype.catch方法是.then(null, rejection)的别名，用于指定发生错误时的回调函数。

```javascript
getJSON('/posts.json').then(function(posts) {
  // ...
}).catch(function(error) {
  // 处理 getJSON 和 前一个回调函数运行时发生的错误
  console.log('发生错误！', error);
});
```

### finally()

finally方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。

```javascript
promise
.then(result => {···})
.catch(error => {···})
.finally(() => {···});
```

### all()

Promise.all方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。

```javascript
const p = Promise.all([p1, p2, p3]);
```

上面代码中，Promise.all方法接受一个数组作为参数，p1、p2、p3都是 Promise 实例，如果不是，就会先调用下面讲到的Promise.resolve方法，将参数转为 Promise 实例，再进一步处理。（Promise.all方法的参数可以不是数组，但必须具有 Iterator 接口，且返回的每个成员都是 Promise 实例。）

p的状态由p1、p2、p3决定，分成两种情况。

（1）只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。

（2）只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时**第一个被reject的实例的**返回值，会传递给p的回调函数。

```javascript
const promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON('/post/' + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});
```

### race()

```javascript
const p = Promise.race([p1, p2, p3]);
```

上面代码中，只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给p的回调函数。

### resolve()

有时需要将现有对象转为 Promise 对象，Promise.resolve方法就起到这个作用。

```javascript
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
```

### reject()

Promise.reject(reason)方法也会返回一个新的 Promise 实例，该实例的状态为rejected。

```javascript
const p = Promise.reject('出错了');
// 等同于
const p = new Promise((resolve, reject) => reject('出错了'))

p.then(null, function (s) {
  console.log(s)
});
```

### 常见错误

**使用其副作用而不是return** 下面的代码有什么问题？

```javascript
somePromise().then(function () {
  someOtherPromise();
}).then(function () {
  // Gee, I hope someOtherPromise() has resolved!
  // Spoiler alert: it hasn't.
});
```

每一个promise对象都会提供一个then方法或者是catch方法

```javascript
somePromise().then(function () {
  // I'm inside a then() function!
});
```

我们在这里能做什么呢？有三种事可以做：

1. 返回另一个promise；

```javascript
getUserByName('nolan').then(function (user) {
  return getUserAccountById(user.id);
}).then(function (userAccount) {
  // I got a user account!
});
```

1. 返回一个同步值（或者undefined)

```javascript
getUserByName('nolan').then(function (user) {
  if (inMemoryCache[user.id]) {
    return inMemoryCache[user.id];    // returning a synchronous value!
  }
  return getUserAccountById(user.id); // returning a promise!
}).then(function (userAccount) {
  // I got a user account!
});
```

函数什么都不返回等于返回了 undefined 目前为止，我们看到给 .then() 传递的都是函数，但是其实它可以接受非函数值：

```javascript
later(1000)
  .then(later(2000))
  .then(function(data) {
    // data = later_1000
  });
```

给 .then() 传递非函数值时，实际上会被解析成 .then(null)，从而导致上一个 promise 对象的结果被“穿透”。于是，上面的代码等价于：

```javascript
later(1000)
  .then(null)
  .then(function(data) {
    // data = later_1000
  });
```

为了避免不必要的麻烦，建议总是给 .then() 传递函数。

1. 抛出一个同步错误。

```javascript
getUserByName('nolan').then(function (user) {
  if (user.isLoggedOut()) {
    throw new Error('user logged out!'); // throwing a synchronous error!
  }
  if (inMemoryCache[user.id]) {
    return inMemoryCache[user.id];       // returning a synchronous value!
  }
  return getUserAccountById(user.id);    // returning a promise!
}).then(function (userAccount) {
  // I got a user account!
}).catch(function (err) {
  // Boo, I got an error!
});
```

**cacth()和then(null, …)并不完全相同**

下面两个代码是不等价的，当使用then(resolveHandler, rejectHandler)，rejectHandler不会捕获在resolveHandler中抛出的错误。

```javascript
somePromise().then(function () {
  return someOtherPromise();
}).catch(function (err) {
  // handle error
});

somePromise().then(function () {
  return someOtherPromise();
}, function (err) {
  // handle error
});
```

对于每个promise对象来说，一旦它被创建，相关的异步代码就开始执行了

**promise坠落现象** 这个错误我在前文中提到的问题中间接的给出了。这个情况比较深奥，或许你永远写不出这样的代码，但是这种写法还是让笔者感到震惊。 你认为下面的代码会输出什么？

```javascript
Promise.resolve('foo').then(Promise.resolve('bar')).then(function (result) {
  console.log(result);
});
```

如果你认为输出的是bar，那么你就错了。实际上它输出的是foo！

产生这样的输出是因为你给then方法传递了一个非函数（比如promise对象）的值，代码会这样理解：then(null)，因此导致前一个promise的结果产生了坠落的效果。你可以自己测试一下：

```javascript
Promise.resolve('foo').then(null).then(function (result) {
  console.log(result);
});
```

让我们回到之前讲解promise vs promise factoriesde的地方。简而言之，如果你直接给then方法传递一个promise对象，代码的运行是和你所想的不一样的。then方法应当接受一个函数作为参数。因此你应当这样书写代码：

```javascript
Promise.resolve('foo').then(function () {
  return Promise.resolve('bar');
}).then(function (result) {
  console.log(result);
});
```

### promise数组依次执行

```javascript
function fetch (api, ms, err = false) {
  return new Promise(function (resolve, reject) {
    console.log(`fetch-${api}-${ms} start`)
    setTimeout(function () {
      err ? reject(`reject-${api}-${ms}`) : resolve(`resolve-${api}-${ms}`)
    }, ms)
  })
}

解法一
function loadData () {
  const promises = [fetch('API1', 3000), fetch('API2', 2000), fetch('API3', 5000)]
  promises.reduce((chain, promise) => {
    return chain.then((res) => {
      console.log(res)
      return promise
    })
  }, Promise.resolve('haha')).then(res => {
    console.log(res)
  })
}

loadData()
// 解法二
async function loadData () {
  const promises = [fetch('API1', 3000), fetch('API2', 2000), fetch('API3', 5000)]
  for (const promise of promises) {
    try {
      await promise.then(res => console.log(res))
    } catch (err) {
      console.error(err)
    }
  }
}
```

### promise常见面试题

1. 

```javascript
const promise = new Promise((resolve, reject) => {
    console.log(1);
    resolve();
    console.log(2);
});
promise.then(() => {
    console.log(3);
});
console.log(4);
```

  输出结果为：1，2，4，3。

  解题思路：then方法是异步执行的。

2. 

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('success')
    reject('error')
  }, 1000)
})
promise.then((res)=>{
  console.log(res)
},(err)=>{
  console.log(err)
})
```

  输出结果：success

  解题思路：Promise状态一旦改变，无法在发生变更。

3. 

```javascript
Promise.resolve(1)
  .then(2)
  .then(Promise.resolve(3))
  .then(console.log)
```

  输出结果：1

  解题思路：Promise的then方法的参数期望是函数，传入非函数则会发生值穿透。

4. 

```javascript
setTimeout(()=>{
  console.log('setTimeout')
})
let p1 = new Promise((resolve)=>{
  console.log('Promise1')
  resolve('Promise2')
})
p1.then((res)=>{
  console.log(res)
})
console.log(1)
```

输出结果：Promise1    1    Promise2    setTimeout

解题思路：这个牵扯到js的执行队列问题，整个script代码，放在了macrotask queue中，执行到setTimeout时会新建一个macrotask queue。但是，promise.then放到了另一个任务队列**microtask queue**中。script的执行引擎会取1个macrotask queue中的task，执行之。然后把所有**microtask queue**顺序执行完，再取setTimeout所在的macrotask queue按顺序开始执行。（具体参考[www.zhihu.com/question/36…](https://link.juejin.im?target=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F36972010)）

```javascript
setImmediate(function(){
    console.log(1);
},0);
setTimeout(function(){
    console.log(2);
},0);
new Promise(function(resolve){
    console.log(3);
    resolve();
    console.log(4);
}).then(function(){
    console.log(5);
});
console.log(6);
process.nextTick(function(){
    console.log(7);
});
console.log(8);
```

结果是：3 4 6 8 7 5 2 1,优先级关系如下：

```javascript
process.nextTick > promise.then > setTimeout > setImmediate
```

V8实现中，两个队列各包含不同的任务：

```javascript
macrotasks: script(整体代码),setTimeout, setInterval, setImmediate, I/O, UI rendering
microtasks: process.nextTick, Promises, Object.observe, MutationObserver
```

**执行过程如下：\**JavaScript引擎首先从macrotask queue中取出\**第一个任务**，执行完毕后，将**microtask queue**中的**所有任务**取出，按顺序**全部执行**；然后再从macrotask queue中取**下一个**，执行完毕后，再次将microtask queue中的全部取出；循环往复，直到两个queue中的任务都取完。

**解释：\**代码开始执行时，所有这些代码在macrotask queue中，取出来执行之。后面遇到了setTimeout，又加入到macrotask queue中，然后，遇到了promise.then，\*\*放入到了另一个队列microtask queue\*\*。等整个execution context stack执行完后，下一步该取的是\**microtask queue**中的任务了。因此promise.then的回调比setTimeout先执行。 

5. 

```javascript
Promise.resolve(1)
    .then((res) => {
        console.log(res);
        return 2;
    })
    .catch((err) => {
        return 3;
    })
    .then((res) => {
        console.log(res);
    });
```

输出结果：1  2

解题思路：Promise首先resolve(1)，接着就会执行then函数，因此会输出1，然后在函数中返回2。因为是resolve函数，因此后面的catch函数不会执行，而是直接执行第二个then函数，因此会输出2。

6. 

```javascript
const promise = new Promise((resolve, reject) => {
setTimeout(() => {
console.log('开始');
resolve('success');
}, 5000);
});
 
const start = Date.now();
promise.then((res) => {
console.log(res, Date.now() - start);
});
 
promise.then((res) => {
console.log(res, Date.now() - start);
});
```

输出结果：

开始

success 5002

success 5002

解题思路：promise 的**.then**或者**.catch**可以被调用多次，但这里 Promise 构造函数只执行一次。或者说 promise 内部状态一经改变，并且有了一个值，那么后续每次调用**.then** 或者**.catch**都会直接拿到该值。

7. 

```javascript
let p1 = new Promise((resolve,reject)=>{
  let num = 6
  if(num<5){
    console.log('resolve1')
    resolve(num)
  }else{
    console.log('reject1')
    reject(num)
  }
})
p1.then((res)=>{
  console.log('resolve2')
  console.log(res)
},(rej)=>{
  console.log('reject2')
  let p2 = new Promise((resolve,reject)=>{
    if(rej*2>10){
      console.log('resolve3')
      resolve(rej*2)
    }else{
      console.log('reject3')
      reject(rej*2)
    }
  })
&emsp;&emsp;return p2
}).then((res)=>{
  console.log('resolve4')
  console.log(res)
},(rej)=>{
  console.log('reject4')
  console.log(rej)
})
```

  输出结果：

    reject1    reject2    resolve3    resolve4    12

  解题思路：我们上面说了Promise的先进之处在于可以在then方法中继续写Promise对象并返回。

8. 

```javascript
new Promise(resolve => {
  console.log(1);
  resolve(3);
  new Promise((resolve2 => {
    resolve2(4)
  })).then(res => {
    console.log(res)
  })
}).then(num => {
  console.log(num)
});
console.log(2)
```

输出1 2 4 3

9. 重头戏！！！！实现一个简单的Promise

```javascript
function Promise(fn){
  var status = 'pending'
  function successNotify(){
      status = 'fulfilled'//状态变为fulfilled
      toDoThen.apply(undefined, arguments)//执行回调
  }
  function failNotify(){
      status = 'rejected'//状态变为rejected
      toDoThen.apply(undefined, arguments)//执行回调
  }
  function toDoThen(){
      setTimeout(()=>{ // 保证回调是异步执行的
          if(status === 'fulfilled'){
              for(let i =0; i< successArray.length;i ++)    {
                  successArray[i].apply(undefined, arguments)//执行then里面的回掉函数
              }
          }else if(status === 'rejected'){
              for(let i =0; i< failArray.length;i ++)    {
                  failArray[i].apply(undefined, arguments)//执行then里面的回掉函数
              }
          }
      })
  }
  var successArray = []
  var failArray = []
  fn.call(undefined, successNotify, failNotify)
  return {
      then: function(successFn, failFn){
          successArray.push(successFn)
          failArray.push(failFn)
          return undefined // 此处应该返回一个Promise
      }
  }
}
```

  解题思路：Promise中的resolve和reject用于改变Promise的状态和传参，then中的参数必须是作为回调执行的函数。因此，当Promise改变状态之后会调用回调函数，根据状态的不同选择需要执行的回调函数。

## async await

ES2017 标准引入了 async 函数，使得异步操作变得更加方便。

async 函数是什么？一句话，它就是 Generator 函数的语法糖。

前文有一个 Generator 函数，依次读取两个文件。

```javascript
var fs = require('fs');

var readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function(error, data) {
      if (error) reject(error);
      resolve(data);
    });
  });
};

var gen = function* () {
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

写成`async`函数，就是下面这样。

```javascript
var asyncReadFile = async function () {
  var f1 = await readFile('/etc/fstab');
  var f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

一比较就会发现，`async`函数就是将 Generator 函数的星号（`*`）替换成`async`，将`yield`替换成`await`，仅此而已。

`async`函数对 Generator 函数的改进，体现在以下四点。

（1）内置执行器。

Generator 函数的执行必须靠执行器，所以才有了`co`模块，而`async`函数自带执行器。也就是说，`async`函数的执行，与普通函数一模一样，只要一行。

```javascript
var result = asyncReadFile();
```

上面的代码调用了`asyncReadFile`函数，然后它就会自动执行，输出最后结果。这完全不像 Generator 函数，需要调用`next`方法，或者用`co`模块，才能真正执行，得到最后结果。

（2）更好的语义。

`async`和`await`，比起星号和`yield`，语义更清楚了。`async`表示函数里有异步操作，`await`表示紧跟在后面的表达式需要等待结果。

（3）更广的适用性。

`co`模块约定，`yield`命令后面只能是 Thunk 函数或 Promise 对象，而`async`函数的`await`命令后面，可以是Promise 对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）。

（4）返回值是 Promise。

`async`函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用`then`方法指定下一步的操作。

进一步说，`async`函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而`await`命令就是内部`then`命令的语法糖。

### 用法

#### 基本用法

`async`函数返回一个 Promise 对象，可以使用`then`方法添加回调函数。当函数执行的时候，一旦遇到`await`就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。

下面是一个例子。

```
async function getStockPriceByName(name) {
  var symbol = await getStockSymbol(name);
  var stockPrice = await getStockPrice(symbol);
  return stockPrice;
}

getStockPriceByName('goog').then(function (result) {
  console.log(result);
});
复制代码
```

上面代码是一个获取股票报价的函数，函数前面的`async`关键字，表明该函数内部有异步操作。调用该函数时，会立即返回一个`Promise`对象。

下面是另一个例子，指定多少毫秒后输出一个值。

```
function timeout(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
}

asyncPrint('hello world', 50);
复制代码
```

上面代码指定50毫秒以后，输出`hello world`。

由于`async`函数返回的是 Promise 对象，可以作为`await`命令的参数。所以，上面的例子也可以写成下面的形式。

```
async function timeout(ms) {
  await new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
}

asyncPrint('hello world', 50);
复制代码
```

async 函数有多种使用形式。

```
// 函数声明
async function foo() {}

// 函数表达式
const foo = async function () {};

// 对象的方法
let obj = { async foo() {} };
obj.foo().then(...)

// Class 的方法
class Storage {
  constructor() {
    this.cachePromise = caches.open('avatars');
  }

  async getAvatar(name) {
    const cache = await this.cachePromise;
    return cache.match(`/avatars/${name}.jpg`);
  }
}

const storage = new Storage();
storage.getAvatar('jake').then(…);

// 箭头函数
const foo = async () => {};
复制代码
```

### 语法

`async`函数的语法规则总体上比较简单，难点是错误处理机制。

#### 返回 Promise 对象

`async`函数返回一个 Promise 对象。

`async`函数内部`return`语句返回的值，会成为`then`方法回调函数的参数。

```
async function f() {
  return 'hello world';
}

f().then(v => console.log(v))
// "hello world"
复制代码
```

上面代码中，函数`f`内部`return`命令返回的值，会被`then`方法回调函数接收到。

`async`函数内部抛出错误，会导致返回的 Promise 对象变为`reject`状态。抛出的错误对象会被`catch`方法回调函数接收到。

```
async function f() {
  throw new Error('出错了');
}

f().then(
  v => console.log(v),
  e => console.log(e)
)
// Error: 出错了
复制代码
```

#### Promise 对象的状态变化

`async`函数返回的 Promise 对象，必须等到内部所有`await`命令后面的 Promise 对象执行完，才会发生状态改变，除非遇到`return`语句或者抛出错误。也就是说，只有`async`函数内部的异步操作执行完，才会执行`then`方法指定的回调函数。

下面是一个例子。

```
async function getTitle(url) {
  let response = await fetch(url);
  let html = await response.text();
  return html.match(/<title>([\s\S]+)<\/title>/i)[1];
}
getTitle('https://tc39.github.io/ecma262/').then(console.log)
// "ECMAScript 2017 Language Specification"
复制代码
```

上面代码中，函数`getTitle`内部有三个操作：抓取网页、取出文本、匹配页面标题。只有这三个操作全部完成，才会执行`then`方法里面的`console.log`。

#### await 命令

正常情况下，`await`命令后面是一个 Promise 对象。如果不是，会被转成一个立即`resolve`的 Promise 对象。

```
async function f() {
  return await 123;
}

f().then(v => console.log(v))
// 123
复制代码
```

上面代码中，`await`命令的参数是数值`123`，它被转成 Promise 对象，并立即`resolve`。

`await`命令后面的 Promise 对象如果变为`reject`状态，则`reject`的参数会被`catch`方法的回调函数接收到。

```
async function f() {
  await Promise.reject('出错了');
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// 出错了
复制代码
```

注意，上面代码中，`await`语句前面没有`return`，但是`reject`方法的参数依然传入了`catch`方法的回调函数。这里如果在`await`前面加上`return`，效果是一样的。

只要一个`await`语句后面的 Promise 变为`reject`，那么整个`async`函数都会中断执行。

```
async function f() {
  await Promise.reject('出错了');
  await Promise.resolve('hello world'); // 不会执行
}
复制代码
```

上面代码中，第二个`await`语句是不会执行的，因为第一个`await`语句状态变成了`reject`。

有时，我们希望即使前一个异步操作失败，也不要中断后面的异步操作。这时可以将第一个`await`放在`try...catch`结构里面，这样不管这个异步操作是否成功，第二个`await`都会执行。

```
async function f() {
  try {
    await Promise.reject('出错了');
  } catch(e) {
  }
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
// hello world
复制代码
```

另一种方法是`await`后面的 Promise 对象再跟一个`catch`方法，处理前面可能出现的错误。

```
async function f() {
  await Promise.reject('出错了')
    .catch(e => console.log(e));
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
// 出错了
// hello world
复制代码
```

#### 错误处理

如果`await`后面的异步操作出错，那么等同于`async`函数返回的 Promise 对象被`reject`。

```
async function f() {
  await new Promise(function (resolve, reject) {
    throw new Error('出错了');
  });
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// Error：出错了
复制代码
```

上面代码中，`async`函数`f`执行后，`await`后面的 Promise 对象会抛出一个错误对象，导致`catch`方法的回调函数被调用，它的参数就是抛出的错误对象。具体的执行机制，可以参考后文的“async 函数的实现原理”。

防止出错的方法，也是将其放在`try...catch`代码块之中。

```
async function f() {
  try {
    await new Promise(function (resolve, reject) {
      throw new Error('出错了');
    });
  } catch(e) {
  }
  return await('hello world');
}
复制代码
```

如果有多个`await`命令，可以统一放在`try...catch`结构中。

```
async function main() {
  try {
    var val1 = await firstStep();
    var val2 = await secondStep(val1);
    var val3 = await thirdStep(val1, val2);

    console.log('Final: ', val3);
  }
  catch (err) {
    console.error(err);
  }
}
复制代码
```

下面的例子使用`try...catch`结构，实现多次重复尝试。

```
const superagent = require('superagent');
const NUM_RETRIES = 3;

async function test() {
  let i;
  for (i = 0; i < NUM_RETRIES; ++i) {
    try {
      await superagent.get('http://google.com/this-throws-an-error');
      break;
    } catch(err) {}
  }
  console.log(i); // 3
}

test();
复制代码
```

上面代码中，如果`await`操作成功，就会使用`break`语句退出循环；如果失败，会被`catch`语句捕捉，然后进入下一轮循环。

#### 使用注意点

第一点，前面已经说过，`await`命令后面的`Promise`对象，运行结果可能是`rejected`，所以最好把`await`命令放在`try...catch`代码块中。

```
async function myFunction() {
  try {
    await somethingThatReturnsAPromise();
  } catch (err) {
    console.log(err);
  }
}

// 另一种写法

async function myFunction() {
  await somethingThatReturnsAPromise()
  .catch(function (err) {
    console.log(err);
  };
}
复制代码
```

第二点，多个`await`命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。

```
let foo = await getFoo();
let bar = await getBar();
复制代码
```

上面代码中，`getFoo`和`getBar`是两个独立的异步操作（即互不依赖），被写成继发关系。这样比较耗时，因为只有`getFoo`完成以后，才会执行`getBar`，完全可以让它们同时触发。

```
// 写法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 写法二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
复制代码
```

上面两种写法，`getFoo`和`getBar`都是同时触发，这样就会缩短程序的执行时间。

第三点，`await`命令只能用在`async`函数之中，如果用在普通函数，就会报错。

```
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  // 报错
  docs.forEach(function (doc) {
    await db.post(doc);
  });
}
复制代码
```

上面代码会报错，因为`await`用在普通函数之中了。但是，如果将`forEach`方法的参数改成`async`函数，也有问题。

```
function dbFuc(db) { //这里不需要 async
  let docs = [{}, {}, {}];

  // 可能得到错误结果
  docs.forEach(async function (doc) {
    await db.post(doc);
  });
}
复制代码
```

上面代码可能不会正常工作，原因是这时三个`db.post`操作将是并发执行，也就是同时执行，而不是继发执行。正确的写法是采用`for`循环。

```
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  for (let doc of docs) {
    await db.post(doc);
  }
}
复制代码
```

如果确实希望多个请求并发执行，可以使用`Promise.all`方法。

```
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = await Promise.all(promises);
  console.log(results);
}

// 或者使用下面的写法

async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = [];
  for (let promise of promises) {
    results.push(await promise);
  }
  console.log(results);
}
复制代码
```

### async 函数的实现原理

async 函数的实现原理，就是将 Generator 函数和自动执行器，包装在一个函数里。

```
async function fn(args) {
  // ...
}

// 等同于

function fn(args) {
  return spawn(function* () {
    // ...
  });
}
复制代码
```

所有的`async`函数都可以写成上面的第二种形式，其中的`spawn`函数就是自动执行器。

下面给出`spawn`函数的实现，基本就是前文自动执行器的翻版。

```
function spawn(genF) {
  return new Promise(function(resolve, reject) {
    var gen = genF();
    function step(nextF) {
      try {
        var next = nextF();
      } catch(e) {
        return reject(e);
      }
      if(next.done) {
        return resolve(next.value);
      }
      Promise.resolve(next.value).then(function(v) {
        step(function() { return gen.next(v); });
      }, function(e) {
        step(function() { return gen.throw(e); });
      });
    }
    step(function() { return gen.next(undefined); });
  });
}
复制代码
```

### 与其他异步处理方法的比较

我们通过一个例子，来看 async 函数与 Promise、Generator 函数的比较。

假定某个 DOM 元素上面，部署了一系列的动画，前一个动画结束，才能开始后一个。如果当中有一个动画出错，就不再往下执行，返回上一个成功执行的动画的返回值。

首先是 Promise 的写法。

```
function chainAnimationsPromise(elem, animations) {

  // 变量ret用来保存上一个动画的返回值
  var ret = null;

  // 新建一个空的Promise
  var p = Promise.resolve();

  // 使用then方法，添加所有动画
  for(var anim of animations) {
    p = p.then(function(val) {
      ret = val;
      return anim(elem);
    });
  }

  // 返回一个部署了错误捕捉机制的Promise
  return p.catch(function(e) {
    /* 忽略错误，继续执行 */
  }).then(function() {
    return ret;
  });

}
复制代码
```

虽然 Promise 的写法比回调函数的写法大大改进，但是一眼看上去，代码完全都是 Promise 的 API（`then`、`catch`等等），操作本身的语义反而不容易看出来。

接着是 Generator 函数的写法。

```
function chainAnimationsGenerator(elem, animations) {

  return spawn(function*() {
    var ret = null;
    try {
      for(var anim of animations) {
        ret = yield anim(elem);
      }
    } catch(e) {
      /* 忽略错误，继续执行 */
    }
    return ret;
  });

}
复制代码
```

上面代码使用 Generator 函数遍历了每个动画，语义比 Promise 写法更清晰，用户定义的操作全部都出现在`spawn`函数的内部。这个写法的问题在于，必须有一个任务运行器，自动执行 Generator 函数，上面代码的`spawn`函数就是自动执行器，它返回一个 Promise 对象，而且必须保证`yield`语句后面的表达式，必须返回一个 Promise。

最后是 async 函数的写法。

```
async function chainAnimationsAsync(elem, animations) {
  var ret = null;
  try {
    for(var anim of animations) {
      ret = await anim(elem);
    }
  } catch(e) {
    /* 忽略错误，继续执行 */
  }
  return ret;
}

复制代码
```

可以看到Async函数的实现最简洁，最符合语义，几乎没有语义不相关的代码。它将Generator写法中的自动执行器，改在语言层面提供，不暴露给用户，因此代码量最少。如果使用Generator写法，自动执行器需要用户自己提供。

### 实例：按顺序完成异步操作

实际开发中，经常遇到一组异步操作，需要按照顺序完成。比如，依次远程读取一组 URL，然后按照读取的顺序输出结果。

Promise 的写法如下。

```
function logInOrder(urls) {
  // 远程读取所有URL
  const textPromises = urls.map(url => {
    return fetch(url).then(response => response.text());
  });

  // 按次序输出
  textPromises.reduce((chain, textPromise) => {
    return chain.then(() => textPromise)
      .then(text => console.log(text));
  }, Promise.resolve());
}

复制代码
```

上面代码使用`fetch`方法，同时远程读取一组 URL。每个`fetch`操作都返回一个 Promise 对象，放入`textPromises`数组。然后，`reduce`方法依次处理每个 Promise 对象，然后使用`then`，将所有 Promise 对象连起来，因此就可以依次输出结果。

这种写法不太直观，可读性比较差。下面是 async 函数实现。

```
async function logInOrder(urls) {
  for (const url of urls) {
    const response = await fetch(url);
    console.log(await response.text());
  }
}

复制代码
```

上面代码确实大大简化，问题是所有远程操作都是继发。只有前一个URL返回结果，才会去读取下一个URL，这样做效率很差，非常浪费时间。我们需要的是并发发出远程请求。

```
async function logInOrder(urls) {
  // 并发读取远程URL
  const textPromises = urls.map(async url => {
    const response = await fetch(url);
    return response.text();
  });

  // 按次序输出
  for (const textPromise of textPromises) {
    console.log(await textPromise);
  }
}

复制代码
```

上面代码中，虽然`map`方法的参数是`async`函数，但它是并发执行的，因为只有`async`函数内部是继发执行，外部不受影响。后面的`for..of`循环内部使用了`await`，因此实现了按顺序输出。

### 异步遍历器

《遍历器》一章说过，Iterator 接口是一种数据遍历的协议，只要调用遍历器对象的`next`方法，就会得到一个对象，表示当前遍历指针所在的那个位置的信息。`next`方法返回的对象的结构是`{value, done}`，其中`value`表示当前的数据的值，`done`是一个布尔值，表示遍历是否结束。

这里隐含着一个规定，`next`方法必须是同步的，只要调用就必须立刻返回值。也就是说，一旦执行`next`方法，就必须同步地得到`value`和`done`这两个属性。如果遍历指针正好指向同步操作，当然没有问题，但对于异步操作，就不太合适了。目前的解决方法是，Generator 函数里面的异步操作，返回一个 Thunk 函数或者 Promise 对象，即`value`属性是一个 Thunk 函数或者 Promise 对象，等待以后返回真正的值，而`done`属性则还是同步产生的。

目前，有一个[提案](https://github.com/tc39/proposal-async-iteration)，为异步操作提供原生的遍历器接口，即`value`和`done`这两个属性都是异步产生，这称为”异步遍历器“（Async Iterator）。

#### 异步遍历的接口

异步遍历器的最大的语法特点，就是调用遍历器的`next`方法，返回的是一个 Promise 对象。

```
asyncIterator
  .next()
  .then(
    ({ value, done }) => /* ... */
  );

复制代码
```

上面代码中，`asyncIterator`是一个异步遍历器，调用`next`方法以后，返回一个 Promise 对象。因此，可以使用`then`方法指定，这个 Promise 对象的状态变为`resolve`以后的回调函数。回调函数的参数，则是一个具有`value`和`done`两个属性的对象，这个跟同步遍历器是一样的。

我们知道，一个对象的同步遍历器的接口，部署在`Symbol.iterator`属性上面。同样地，对象的异步遍历器接口，部署在`Symbol.asyncIterator`属性上面。不管是什么样的对象，只要它的`Symbol.asyncIterator`属性有值，就表示应该对它进行异步遍历。

下面是一个异步遍历器的例子。

```
const asyncIterable = createAsyncIterable(['a', 'b']);
const asyncIterator = asyncIterable[Symbol.asyncIterator]();

asyncIterator
.next()
.then(iterResult1 => {
  console.log(iterResult1); // { value: 'a', done: false }
  return asyncIterator.next();
})
.then(iterResult2 => {
  console.log(iterResult2); // { value: 'b', done: false }
  return asyncIterator.next();
})
.then(iterResult3 => {
  console.log(iterResult3); // { value: undefined, done: true }
});

复制代码
```

上面代码中，异步遍历器其实返回了两次值。第一次调用的时候，返回一个 Promise 对象；等到 Promise 对象`resolve`了，再返回一个表示当前数据成员信息的对象。这就是说，异步遍历器与同步遍历器最终行为是一致的，只是会先返回 Promise 对象，作为中介。

由于异步遍历器的`next`方法，返回的是一个 Promise 对象。因此，可以把它放在`await`命令后面。

```
async function f() {
  const asyncIterable = createAsyncIterable(['a', 'b']);
  const asyncIterator = asyncIterable[Symbol.asyncIterator]();
  console.log(await asyncIterator.next());
  // { value: 'a', done: false }
  console.log(await asyncIterator.next());
  // { value: 'b', done: false }
  console.log(await asyncIterator.next());
  // { value: undefined, done: true }
}

复制代码
```

上面代码中，`next`方法用`await`处理以后，就不必使用`then`方法了。整个流程已经很接近同步处理了。

注意，异步遍历器的`next`方法是可以连续调用的，不必等到上一步产生的Promise对象`resolve`以后再调用。这种情况下，`next`方法会累积起来，自动按照每一步的顺序运行下去。下面是一个例子，把所有的`next`方法放在`Promise.all`方法里面。

```
const asyncGenObj = createAsyncIterable(['a', 'b']);
const [{value: v1}, {value: v2}] = await Promise.all([
  asyncGenObj.next(), asyncGenObj.next()
]);

console.log(v1, v2); // a b

复制代码
```

另一种用法是一次性调用所有的`next`方法，然后`await`最后一步操作。

```
const writer = openFile('someFile.txt');
writer.next('hello');
writer.next('world');
await writer.return();

复制代码
```

#### for await...of

前面介绍过，`for...of`循环用于遍历同步的 Iterator 接口。新引入的`for await...of`循环，则是用于遍历异步的 Iterator 接口。

```
async function f() {
  for await (const x of createAsyncIterable(['a', 'b'])) {
    console.log(x);
  }
}
// a
// b

复制代码
```

上面代码中，`createAsyncIterable()`返回一个异步遍历器，`for...of`循环自动调用这个遍历器的`next`方法，会得到一个Promise对象。`await`用来处理这个Promise对象，一旦`resolve`，就把得到的值（`x`）传入`for...of`的循环体。

`for await...of`循环的一个用途，是部署了 asyncIterable 操作的异步接口，可以直接放入这个循环。

```
let body = '';
for await(const data of req) body += data;
const parsed = JSON.parse(body);
console.log('got', parsed);

复制代码
```

上面代码中，`req`是一个 asyncIterable 对象，用来异步读取数据。可以看到，使用`for await...of`循环以后，代码会非常简洁。

如果`next`方法返回的Promise对象被`reject`，那么就要用`try...catch`捕捉。

```
async function () {
  try {
    for await (const x of createRejectingIterable()) {
      console.log(x);
    }
  } catch (e) {
    console.error(e);
  }
}

复制代码
```

注意，`for await...of`循环也可以用于同步遍历器。

```
(async function () {
  for await (const x of ['a', 'b']) {
    console.log(x);
  }
})();
// a
// b

复制代码
```

#### 异步Generator函数

就像 Generator 函数返回一个同步遍历器对象一样，异步 Generator 函数的作用，是返回一个异步遍历器对象。

在语法上，异步 Generator 函数就是`async`函数与 Generator 函数的结合。

```
async function* readLines(path) {
  let file = await fileOpen(path);

  try {
    while (!file.EOF) {
      yield await file.readLine();
    }
  } finally {
    await file.close();
  }
}

复制代码
```

上面代码中，异步操作前面使用`await`关键字标明，即`await`后面的操作，应该返回Promise对象。凡是使用`yield`关键字的地方，就是`next`方法的停下来的地方，它后面的表达式的值（即`await file.readLine()`的值），会作为`next()`返回对象的`value`属性，这一点是于同步Generator函数一致的。

可以像下面这样，使用上面代码定义的异步Generator函数。

```
for await (const line of readLines(filePath)) {
  console.log(line);
}

复制代码
```

异步 Generator 函数可以与`for await...of`循环结合起来使用。

```
async function* prefixLines(asyncIterable) {
  for await (const line of asyncIterable) {
    yield '> ' + line;
  }
}

复制代码
```

`yield`命令依然是立刻返回的，但是返回的是一个Promise对象。

```
async function* asyncGenerator() {
  console.log('Start');
  const result = await doSomethingAsync(); // (A)
  yield 'Result: '+ result; // (B)
  console.log('Done');
}

复制代码
```

上面代码中，调用`next`方法以后，会在`B`处暂停执行，`yield`命令立刻返回一个Promise对象。这个Promise对象不同于`A`处`await`命令后面的那个 Promise 对象。主要有两点不同，一是`A`处的Promise对象`resolve`以后产生的值，会放入`result`变量；二是`B`处的Promise对象`resolve`以后产生的值，是表达式`'Result： ' + result`的值；二是`A`处的 Promise 对象一定先于`B`处的 Promise 对象`resolve`。

如果异步 Generator 函数抛出错误，会被 Promise 对象`reject`，然后抛出的错误被`catch`方法捕获。

```
async function* asyncGenerator() {
  throw new Error('Problem!');
}

asyncGenerator()
.next()
.catch(err => console.log(err)); // Error: Problem!

复制代码
```

注意，普通的 async 函数返回的是一个 Promise 对象，而异步 Generator 函数返回的是一个异步Iterator对象。基本上，可以这样理解，`async`函数和异步 Generator 函数，是封装异步操作的两种方法，都用来达到同一种目的。区别在于，前者自带执行器，后者通过`for await...of`执行，或者自己编写执行器。下面就是一个异步 Generator 函数的执行器。

```
async function takeAsync(asyncIterable, count=Infinity) {
  const result = [];
  const iterator = asyncIterable[Symbol.asyncIterator]();
  while (result.length < count) {
    const {value,done} = await iterator.next();
    if (done) break;
    result.push(value);
  }
  return result;
}

复制代码
```

上面代码中，异步Generator函数产生的异步遍历器，会通过`while`循环自动执行，每当`await iterator.next()`完成，就会进入下一轮循环。

下面是这个自动执行器的一个使用实例。

```
async function f() {
  async function* gen() {
    yield 'a';
    yield 'b';
    yield 'c';
  }

  return await takeAsync(gen());
}

f().then(function (result) {
  console.log(result); // ['a', 'b', 'c']
})

复制代码
```

异步 Generator 函数出现以后，JavaScript就有了四种函数形式：普通函数、async 函数、Generator 函数和异步 Generator 函数。请注意区分每种函数的不同之处。

最后，同步的数据结构，也可以使用异步 Generator 函数。

```
async function* createAsyncIterable(syncIterable) {
  for (const elem of syncIterable) {
    yield elem;
  }
}

复制代码
```

上面代码中，由于没有异步操作，所以也就没有使用`await`关键字。

#### yield* 语句

`yield*`语句也可以跟一个异步遍历器。

```
async function* gen1() {
  yield 'a';
  yield 'b';
  return 2;
}

async function* gen2() {
  const result = yield* gen1();
}

复制代码
```

上面代码中，`gen2`函数里面的`result`变量，最后的值是`2`。

与同步Generator函数一样，`for await...of`循环会展开`yield*`。

```
(async function () {
  for await (const x of gen2()) {
    console.log(x);
  }
})();
// a
// b
```


作者：Jre
链接：https://juejin.im/post/5b0399bc518825426f311a3e
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



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

```javascript
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

## Js异步执行顺序

今天在重温promise的时候，突然发现了一个有趣的题目：

```js
(function() {
    setTimeout(() => {
        console.log(0);
    });
    new Promise(resolve => {
        console.log(1);
        setTimeout(() => {
            resolve();
            Promise.resolve().then(() => {
                console.log(2);
                setTimeout(() => console.log(3));
                Promise.resolve().then(() => console.log(4));
            });
        });
        Promise.resolve().then(() => console.log(5));
    }).then(() => {
        console.log(6);
        Promise.resolve().then(() => console.log(7));
        setTimeout(() => console.log(8));
    });
    console.log(9);
})();
```

我们先放上结果：

```js
1 9 5 0 6 2 7 4 8 3
```

### 简要讲述

首先我们知道JS分为了同步和异步，并且是顺序是先同步后异步，也就是同步代码执行完成后，哪怕异步代码到了它执行的时候，也会先让同步执行完。但是JS的执行顺序还可以分得更细，那就是异步的顺序。 我们知道很多异步任务，比如：

```js
setTimeout, setInterval, setImmediate, Promises.then, Promise.catch
```

甚至还有nodejs中的process.nextTick等，就像我们知道process.nextTick比promise先执行一样。那么这些异步事件是谁先执行谁后执行呢。 这涉及到了事件循环(event loop)。事件循环也就是事件出入栈。 上面说了这么多异步任务，我们先把它们分为两类： 宏任务（macro-task）：setTimeout, setInterval, setImmediate 微任务（micro-task）：Promises.then, Promise.catch

然后我们再明白一个概念：先执行微任务，再执行宏任务。这种执行是，先执行清空微任务队列，再执行宏任务，当当前宏任务执行完后，微任务队列中又有微任务，再次将微任务队列执行完，再执行下一个宏任务。

在异步任务执行过程中遇到宏任务与微任务，将其依次放入当前事件循环队列中。

### 例题描述

首先执行主线程，也就是script（script是宏任务），因为此时没有微任务

然后遇到`setTimeout(() => { console.log(0);});` 这是宏任务，将其放入宏任务队列中 记为 -- 宏任务1

promise新建后会立即执行，输出`1`

然后将

```js
        setTimeout(() => {
            resolve();
            Promise.resolve().then(() => {
                console.log(2);
                setTimeout(() => console.log(3));
                Promise.resolve().then(() => console.log(4));
            });
        });
```

继续放入宏任务队列中 记为 -- 宏任务2

将`Promise.resolve().then(() => console.log(5));`放入微任务队列中 记为 -- 微任务1

继续往下走，输出`9`

此时当前宏任务执行完毕，开始执行微任务队列，目前微任务队列中只有微任务1，执行，输出`5`

微任务队列此时清空，然后执行宏任务1，输出`0`

微任务队列依然没有任务，继续执行宏任务2，遇见 `resolve();`，于是将

```js
.then(() => {
        console.log(6);
        Promise.resolve().then(() => console.log(7));
        setTimeout(() => console.log(8));
    });
    console.log(9);
})
```

放入微任务队列中，记为 -- 微任务2，再将

```js
Promise.resolve().then(() => {
      console.log(2);
      setTimeout(() => console.log(3));
      Promise.resolve().then(() => console.log(4));
});
```

记为 -- 微任务3

当前宏任务执行完毕，开始执行微任务，先执行微任务2，输出`6`；将`Promise.resolve().then(() => console.log(7));`记为 -- 微任务4；`setTimeout(() => console.log(8));`记为 -- 宏任务3

执行微任务3，输出`2`；将`setTimeout(() => console.log(3));`记为 -- 宏任务4；将`Promise.resolve().then(() => console.log(4));`记为 -- 微任务5；

此时队列中依然有微任务，执行微任务4，输出`7`；执行微任务5，输出`4`；微任务队列执行完毕

执行宏任务3，输出`8`；执行宏任务4，输出`3`；全部执行完毕

那么我们的结果为`1 9 5 0 6 2 7 4 8 3`



