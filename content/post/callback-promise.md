---
date: 2017-04-05
title: 理解 Javascript 的几种异步模式
tags: ["Javascript"]
categories: ["DevOps"]
---

Javacript 提供了基于回调的异步编程模式，如回调函数，Promise对象，Async Function 等。下面是我在学习过程中总结的
一些特性和使用方式。

## Javascript 运行时概念

![js runtime](/static/callback-promise/js-runtime-concept.svg)

* 栈(Stack)

和其他语言一样, 栈保存了函数调用的层次关系, 栈中的每一块叫做一帧(frame), 最上层的那一帧, 代表了最内层的函数,
当函数返回时, 清空帧, 当调用栈中所有的帧清空(即最外层函数返回), 当前执行的代码块结束。
JS调用栈有三个特点:

>	1. 单线程, 浏览器的JS引擎本身是多线程的, 但是JS的调用栈是单线程的, 一次只能执行一段代码;
>	2. 同步执行, 在栈中的任务依次执行并返回, 不能在任务之间切换
>	3. 非阻塞, 在线程繁忙时, 浏览器仍然可以接收事件

* 堆(Heap)

堆中保存了大量的对象,垃圾收集器定期清理未被引用的对象。

* 事件队列(event queue)

当调用某些Web APIs时,如DOM事件(如鼠标点击操作)、XMLHttpRequest, setTimeout()等,
会添加callback函数到event queue。

* Event loop

Event loop将event queue中按照"FIFO"的顺序将callback取出,放入栈中,callback函数返回,清空栈,
循环执行上述步骤。

MDN提供的伪代码实现如下:
```
while(queue.waitForMessage()) {
	queue.processNextMessage();
}
```

---

## Callback

### Callback函数
和其他编程语言(Java, Ruby)不同,在JS中,函数也是对象(`Function instanceof Object`返回为`true`)。
函数是JS的"一等公民",通俗的理解是,可以将函数作为参数传给另外一个函数,也可以将一个函数作为另一个函数的返回值。

```javascript
function bar(cb) {
	setTimeout(cb,2000);
}

function foo() {
	console.log("hehe")
}

bar(foo)
```

上面的代码中,foo就是callback函数,上述代码的执行过程大致如下图所示:

![event queue](/static/callback-promise/event-queue.svg)

### Callback hell

思考如下代码:
```javascript
listen( "click", function handler(evt){
	setTimeout( function request(){
		ajax( "http://some.url.1", function response(text){
			if (text == "hello") {
				handler();
			}
			else if (text == "world") {
				request();
			}
		} );
	}, 2000) ;
} );
```

这样的代码常常被称作"callback hell",也被称为"倒金字塔"。
但是callback hell,不止看上去的缩进/嵌套问题。代码层面的问题,
可以通过重构,将回调函数作为参数传递解决:

```javascript
// step 1
listen( "click", handler );

// step 2
function handler() {
	setTimeout( request, 500 );
}

// step 3
function request(){
	ajax( "http://some.url.1", response );
}

// step 4
function response(text){
	if (text == "hello") {
		handler();
	}
	else if (text == "world") {
		request();
	}
}
```

这样做的结果很明显,没有了大段的嵌套代码,但依然有很多问题没有解决:

1. 数据流出现在多个函数之间,需要跳转多个函数才能看清楚, 
当代码结构变得复杂,清晰的数据流更难以获取;
2. 各个步骤之间通过hard code进行连接,无法复用;
3. 在每个步骤都要对异常进行处理,产生逻辑重复的代码等;

**Last but most important**, callback 将控制权交给了第三方,由此产生一系列的信任问题,
大部分情况下,我们不需要过多的考虑信任问题(如过早/过晚回调、多次回调等),但是也只是还没有出现问题而已。

---

## Promise

### 什么是Promise

Promise并不是Javascript独有的,在多种编程语言中都存在类似的概念如Java/Scala中的Future,
在1976年前后已经陆续提出了这些概念。

MDN的描述:

> A Promise is a proxy for a value not necessarily known when the promise is created

Promise/A+:

> A promise represents the eventual result of an asynchronous operation.

我的理解:

> Promise对异步/同步操作进行动态代理, 并通过规定的 `then` 函数访问异步/同步操作的结果, 访问的过程是异步(通过回调函数)的。
由于 chainable、immutable、规范化的异常处理等特点, 使异步代码符合人的思考方式, 弥补了回调函数的一些不足(如 callback hell, 多次回调等)。
但是从实现来看, Promise 的根本还是回调, 只是通过巧妙标准的API简化了整个流程。

### Promise API

Promise API流程如下:

![Promise API flow](/static/callback-promise/promise.png)

#### Promise状态

- pending: 初始状态
- fulfilled: 成功状态
- rejected: 失败状态

#### Promise方法

```javascript
Promise.all([promise1, promise2, …]);
Promise.race([promise1, promise2, …]);
Promise.reject(value);
Promise.resolve(value);
Promise.catch(onRejection);
Promise.then(onFulFillment, onRejection);
```

### Simple Example

#### 构造一个 Promise, 通过 `then` 函数取值:

```javascript
var myPromise = new Promise((resolve, reject) => {
    setTimeout(function() {
        resolve("success");
    }, 3000);
});

myPromise.then((msg) => {
   console.log("Yay, " + msg);
});

console.log("i am not promise");
```

Console输出:
```
i am not promise
Yay, success
```

#### `then` 的回调函数有返回值时,返回一个 relove 该值的 Promise:
```javascript
var myPromise = new Promise((resolve, reject) => {
    setTimeout(function() {
        resolve("success");
    }, 3000);
});

var myPromise2 = myPromise.then((msg) => {
   console.log("Yay, " + msg);
   return "success 2";
});

myPromise2.then((msg) => {
    console.log("Hey, " + msg);
});

console.log("i am not promise");
```
Console输出:
```
i am not promise
Yay, success
Hey, success 2
```

#### `then`的回调函数中没有返回值,调用`then`函数返回一个Promise,其relove的值为`undefined`
```javascript
var myPromise = new Promise((resolve, reject) => {
    setTimeout(function() {
        resolve("success");
    }, 3000);
});

var myPromise2 = myPromise.then((msg) => {
   console.log("Yay, " + msg);
});

myPromise2.then((msg) => {
    console.log("Hey, " + msg);
});

console.log("i am not promise");
```
Console输出:
```
i am not promise
Yay, success
Hey, undefined
```

#### 同一个Promise被多次使用时,执行顺序为`then`调用的先后顺序:
```javascript
var myPromise = new Promise((resolve, reject) => {
    setTimeout(function() {
        resolve("success");
    }, 3000);
});

myPromise.then((msg) => {
   console.log("Yay, " + msg);
});

myPromise.then((msg) => {
    console.log("Hey, " + msg);
})

myPromise.then((msg) => {
    console.log("bazhahei, " + msg);
})
console.log("i am not promise");
```
Console输出
```
i am not promise
Yay, success
Hey, success
bazhahei, success
```

#### Promise.all (异步执行)

```javascript
var p1 = Promise.resolve('haha');
var p2 = Promise.resolve('heheda');

var p3 = Promise.all([p1,p2]);
p3.then((msg)=>{
    console.log(p3);
    console.log(msg);
});
console.log(p3)
var p2_reject = Promise.reject('err');

var p4 = Promise.all([p1,p2_reject]);
p4.then(null,(msg)=>console.log(msg));
```

Console输出
```
Promise {[[PromiseStatus]]: "pending", [[PromiseValue]]: undefined}
Promise {[[PromiseStatus]]: "resolved", [[PromiseValue]]: Array[2]}
["haha", "heheda"]
err
```

#### Promise.race (异步执行)

```javascript
var p1= Promise.reject('haha')
console.log("didi")
var p2 = Promise.resolve('heheda')
var p3 = Promise.race([p1,p2]).then(
    (msg)=>console.log(msg), (reason)=>console.log(reason)
)

console.log(p3)
console.log("over")
```
Console输出
```
Promise {[[PromiseStatus]]: "pending", [[PromiseValue]]: undefined}
over
haha
```

### Promise 实现
1. 一个简易的Promise的实现参考[这里](https://gist.github.com/unscriptable/814052)
2. http://www.mattgreer.org/articles/promises-in-wicked-detail/
3. [cujojs/when](https://github.com/cujojs/when)

---

## Generator
Generator函数是一种特殊的函数,它不是 `run to completion` 的,而是可以多次分段执行,
通过双向消息传递(2-way messsage passing)实现状态机、异步流程控制等多种功能。

调用generator函数可以返回一个generator对象（既是iterator也是iterable）。
注：

> Iterator: 一个实现了next函数的对象，next返回返回值为`{value: xxx, done: true/false}`
  Iterable: 一个含有属性为`Symbol.iterator`的对象，`Symbol.iterator`是一个可以返回上述Iterator对象的函数。

### Simple example
`yield` 使用：
```javascript
function* generator() {
  yield 1;
  yield 2;
  yield 3;
}
var g1 = generator()
console.log(g1.next())
console.log(g1.next())
console.log(g1.next())
console.log(g1.next())
```
console 输出
```
{value: 1, done: false}
{value: 2, done: false}
{value: 3, done: false}
{value: undefined, done: true}
```
`yield*`使用,将控制权交给另外一个Iterable对象：
```javascript
function* generator() {
  yield 1;
  yield* [2.1,2.2,2.3];
  yield 3;
}
var g2 = generator()
console.log(g2.next())
console.log(g2.next())
console.log(g2.next())
console.log(g2.next())
console.log(g2.next())
console.log(g2.next())
```
输出：
```
{value: 1, done: false}
{value: 2.1, done: false}
{value: 2.2, done: false}
{value: 2.3, done: false}
{value: 3, done: false}
{value: undefined, done: true}
```
上面两个例子，只用到了单向消息传递，即只关注`next()`的返回值，下面让我举一个双向消息传递的栗子：
```javascript
function* twoWayMessagePassing() {
  let a = yield 1;
  let b = yield 2;
  return a + b;
}

var g3 = twoWayMessagePassing()
console.log(g3.next());
console.log(g3.next(3));
console.log(g3.next(4));
```
输出：
```
{value: 1, done: false}
{value: 2, done: false}
{value: 7, done: true}
```
### 理解双向消息传递

根据上述第三个例子，画图如下,
每一次`next`函数执行一段代码块时，将参数赋给代码块开始处的yield表达式(如果有的话)，并将代码块结束处的yield/return表达式的值作为`next`函数的返回值。

![two-way-message-passing](/static/callback-promise/two-way-message-passing.svg)

### 其他API
```javascript
Generator.prototype.return()
Generator.prototype.throw()
```

### Generator结合Promise实现异步控制流

一个简易的控制方法如下，需要 **手动迭代**：
```javascript
function foo(a,b) {
    return new Promise((resolve)=>{
        setTimeout(()=>{
            resolve(a+b);
        },2000);
    })
}

function* gen() {
    try {
        var sum1 = yield foo(3, 6);
        var sum2 = yield foo(7, 9);
        return sum1 + sum2;
    } catch (err) {
        console.log(err);
    }
}

var it = gen();
var p = it.next().value

p.then((msg)=>{
    return it.next(msg).value
}).then((msg)=>{
    return it.next(msg).value
}).then((msg)=>{
    console.log(msg)
})
```
输出：`25`。

修改上述过程实现 **自动迭代**：
```javascript
var isPromise = (obj) => typeof obj !== 'undefined' &&
  typeof obj.then === 'function';

var next = (iter, callback, prev = undefined) => {
  const item = iter.next(prev);
  const value = item.value;

  if (item.done) return callback(prev);

  if (isPromise(value)) {
    value.then(val => {
      setTimeout(() => next(iter, callback, val),0);
    });
  } else {
    setTimeout(() => next(iter, callback, value),0);
  }
};

var gensync = (fn) => {
  return (...args) =>  {
    return new Promise(resolve => {
      next(fn(...args), val => resolve(val));
    })
  }
};

var foo = (a, b) => new Promise((resolve) => {
  setTimeout(()=>{
      resolve(a+b);
  },2000);
});

var asyncFunc = gensync(function* () {
  var result1 = yield foo(3, 6); // returns promise
  var result2 = yield foo(7, 9) // returns another promise
  // waits for promise and uses promise result
  yield result1 + result2;
});

asyncFunc()
  .then(val => console.log(val));
```
输出：`25`。

---

## Async Function (ES7)
Async Function可以将异步流程以顺序执行的方式来描述，增强了可读性，其实就是上述Generator和Promise结合的 _语法糖_
上述代码可以写成以下形式：
```javascript
function foo(a,b) {
    return new Promise((resolve)=>{
        setTimeout(()=>{
            resolve(a+b);
        }, 2000);
    })
}

async function sum() {
    var sum1 = await foo(3, 6);
    var sum2 = await foo(7, 9);
    return sum1 + sum2;
}

sum().then(v => {console.log(v)})
```
输出: `25`。

`await` 表达式"暂停"了async函数的执行，等待某个Promise变为fulfilled/rejected的状态，
此时，再继续执行async函数后面的代码块，最终返回一个Promise对象。

由此，上述async函数`sum`可以优化为：
```javascript
async function sum() {
    var sum1 = foo(3, 6);
    var sum2 = foo(7, 9);
    return await sum1 + await sum2;
}
```

## 总结
Javascript越来越广泛的应用于Web前后端的开发工作之中，合理使用JS的异步编程模式，可以提高程序的性能，增加代码的可维护性和可读性。

## Reference
[1] https://developer.mozilla.org/en/docs/Web/JavaScript/EventLoop  
[2] [How is javascript asynchronous AND single threaded?](http://www.sohamkamani.com/blog/2016/03/14/wrapping-your-head-around-async-programming/)  
[3] [You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS)  
[4] [ECMAScript® 2017 Language Specification](https://tc39.github.io/ecma262/)  
[5] http://www.mattgreer.org/articles/promises-in-wicked-detail/  
[6] [Promise/A+](https://promisesaplus.com)  
[7] [Iteration protocol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)  
