---
title: ' javascript 中的事件循环 Event Loop'
date: 2020-07-09 09:05:16
tags: js
---

## javascript 为什么是单线程?

单线程也就是同一时间只能做一件事, 这也是js 语言的特点,那么为什么不弄个多线程呢?

js的单线程与它的用于有关,作为浏览器语言, js,做的是与用户互动,操作DOM,如果是多线程了, 同一时间对一个DOM进行了增加和删除操作,这时候以哪一个为准?

为了利用多核CPU的计算能力，HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以，这个新标准并没有改变JavaScript单线程的本质

可以看下阮一峰老师的博客


## 同步与异步

### 同步

看下面代码
这段代码的实现就叫做同步,也就是说按照顺序去做,做完第一件事情之后,再去做第二件事情

```
console.log(1)
for(let i = 0; i< 3; i++){
  console.log(i)
}
console.log(4)
```
### 异步 (细分为宏任务和微任务)

因为js是单线程的, 当任务多的时候就需要排队, 如果前面一个任务耗时很长, 后面一个任务就要一直等待.所以异步就出现了

宏任务有以下几种：
①I/O
②setTimeout
③setInterval
④setImmediate
⑤requestAnimationFrame
微任务有以下几种：
①process.nextTick
②MutationObserver
③Promise.then catch finally

```
console.log('start') // 同步

setTimeout(() => {
  console.log('宏任务')
},0)

new Promise((reslove) => {
  console.log('p1') // prpmise 创建 就会立即执行
  reslove()
}).then(res => {
  console.log('p2') // 
})

同步 > 微任务 > 宏任务 
```

```
console.log('start') // 同步

setTimeout(() => {
  console.log('宏任务')
},2220)

for (let index = 0; index < 22000; index++) {
console.log('index')  // setTimeout 的延时时间,取决与同步函数的运行时间, 这里登 延时函数运行的时候会立即输出
}


new Promise((reslove) => {
  console.log('p1') // prpmise 创建 就会立即执行
  reslove()
}).then(res => {
  console.log('then') // 
})

```
这个和上面的差不多, 只要理解了一个就面的也就理解了, 

```
console.log('start') // 同步

setTimeout(() => {
  console.log('宏任务')
  new Promise((reslove) => {
    console.log('setTimeout p1') // prpmise 创建 就会立即执行
    reslove()
  }).then(res => {
    console.log(' setTimeout then') // 
  })
}, 2220)

for (let index = 0; index < 22000; index++) {
  console.log('index')
}


new Promise((reslove) => {
  console.log('p1') // prpmise 创建 就会立即执行
  reslove()
}).then(res => {
  console.log('then') // 
})

```
记住JS是单线程的, 任务也是一个一个取的

```
let i = 0

setTimeout(() => { // 扔到任务队列 => 依次执行 i = 1
  console.log(i++)
},1000)

setTimeout(() => {  // 扔到任务队列 => 依次执行 i = 2
  console.log(i++)
},1000)

```
## 任务队列(消息队列)
任务队列是一个事件的队列(也可以理解成消息的队列)工作线程完成一项任务，就在"任务队列"中添加一个事件(也可以理解为发送一条消息)，表示相关的异步任务可以进入"执行栈"了。主线程读取"任务队列"，就是读取里面有哪些事件。

- 所有同步任务都在主线程上执行，形成一个执行栈
- 主线程发起异步请求,相应的工作线程就会去执行异步任务
  
主线程可以继续执行后面的代码

- 主线程之外，还存在一个"任务队列"（task queue）。只要异步任务
- 一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队
- 主线程把当前的事件执行完成之后,再去读取任务队列,如此反复重复


## 总结  

主线程从"任务队列"中读取事件，这个过程是循环不断的，所以整个的这种运行机制又称为Event Loop（事件循环）

代码从上到下执行, 优先执行同步函数, 在遇到异步函数时将该任务推入执行栈,当任务队列中没有同步任务,便开始从执行栈中取异步函数, 顺序是 微任务 > 宏任务 

