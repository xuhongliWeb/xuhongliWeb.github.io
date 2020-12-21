---
title: Promise
date: 2020-12-20 10:25:09
tags: js
---

## Promise 是什么？
Promise 是异步编程的一种解决方案，比传统的方案一一回调更合理更强大。 

所谓Promise ,简单来说。里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果从语法上说 。 Promise 是一个对象， 从它可以获取异步操作的消息。Promise提供的API，各种异步操作都可以用通用的方法处理 。

Promise 对象有两个特点

- 对象的状态不受外界影响Promise对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态
- 一旦状态改变。就不会在变

Promise也有一些缺点。首先，无法取消Promise，一旦新建它就会立即执行，无法中途取消。其次，如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。第三，当处于pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。

## 基本语法

* promise.prototype.then()
* Promise.prototype.catch() 
  promise.prototype.then(res,catch) 的第二个参数也能捕捉到错误，一般不推荐
* Promise.prototype.finally()
  finally方法的回调函数不接受任何参数，这意味着没有办法知道，前面的 Promise 状态到底是fulfilled还是rejected
* Promise.all() 
  Promise.all()方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。
  只有所有all里面包含的promise状态为fulfilled 整体才会返回fulfilled, 同样一个错误整体就返回rejected了
* promise.race()
  Promise.race()方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例。
  ```
  const p = Promise.race([p1, p2, p3]);
  ```
  上述代码 p1,p2,p3 只要有一个实列率先改变， p的状态就跟着改变。 那个率先改变的promise返回值就传递给离p的回调函数 
* const p = Promise.race([p1, p2, p3]); 
  Promise.allSettled()方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。只有等到所有这些参数实例都返回结果，不管是fulfilled还是rejected，包装实例才会结束。该方法由 ES2020 引入。
* Promise.resolve() 
  有时需要将现有对象转为 Promise 对象，Promise.resolve()方法就起到这个作用。
  ```
  Promise.resolve('foo')
  // 等价于
  new Promise(resolve => resolve('foo'))
  ```
* Promise.reject()
  Promise.reject(reason)方法也会返回一个新的 Promise 实例，该实例的状态为rejected。


## 手写基本Promise

这一版本 支持 
* 异步任务
* then
* catch
简版 Promise

```

/**
 * @description 
 * executor - 传进来的一堆 用户写的方法
 * promise 对象初始状态为peading, 在被resolve或 reject 是 状态改为 Fulfilled 或 Rejected
 * resolve 接收成功的数据， reject 接收失败或错误的数据
 * Promise对象必须有一个then方法， 且只接收两个可函数参数 onFulfilled / onRejected
 * 发布订阅模式支持异步
 * catch方法其实就是没有成功回调的then方法，这个很好理解，因为一旦失败之后就会调用reject,最终都会走到then方法的失败回调中，只是简单的把then方法换个名字而已。
 */
 const pConfig = {
   PEADINGD : 'PEADINGD', 
   RESOLVED: 'RESOLVED',
   REJECTED: 'RESOLVED'
 }
 export class vxPromise {
   constructor(executor) {
    console.log(executor, '执行者 executor -1 ')

    this.status = pConfig.PEADINGD // 默认等待状态
    this.value = undefined // 这个应该是 成功的结果把？
    this.reason = undefined // 失败的结果？
    this.onResolvedCallbacks = [] // 存放 成功函数的
    this.onRejectedCallbacks = [] // 存放失败函数的
    let resolve = (value) => {
      console.log(value, 'resolve-value -4')
      if(this.status === pConfig.PEADINGD) { // 等待状态才执行
        this.value = value 
        this.status = pConfig.RESOLVED
        // 依次执行异步任务 由 then 的时候push 进去
        this.onResolvedCallbacks.forEach(fn => {
          console.log(fn,'回调里的', '成功')
          // fn()
        })
      }
    }
    let reject = (value) => {
      console.log(value, 'reject-value-4')
      if(this.status === pConfig.PEADINGD) {
        this.reason = value // 更新状态
        this.status = pConfig.REJECTED
        // 依次执行异步任务 由 then 的时候push 进去
        this.onRejectedCallbacks.forEach(fn => {
          console.log(fn,'回调里的', '失败')
          // fn()
        })
      }
    }
    // 执行 executor 传入我们定义的成功和失败的函数； 把用户写的resolve,reject的结果 传入 内部  resolve 和 reject 
    try {
      executor(resolve,reject)
    }catch (e) {
      console.log('catch错误', e);
			reject(e); //如果内部出错 直接将error手动调用reject向下传递
    }

   }
   then(onfufilled,onrejected) {
     console.log( this.onResolvedCallbacks, 'onrejected-2')
    if(this.status === pConfig.RESOLVED) {
      onfufilled(this.value)
    }
    if(this.status === pConfig.REJECTED) {
      onrejected(this.reason)
    }
    // 执行异步任务
    if(this.status === pConfig.PEADINGD) {
      console.log('push-3')
      this.onResolvedCallbacks.push(() => {
        console.log('55')
        onfufilled(this.value);
      })
      this.onRejectedCallbacks.push(() => {
        console.log('55')
				onrejected(this.reason);
			});
    }
    
  }
  catch(errCallback) { 
    return this.then(null, errCallback);
  }
 }
 ```

完整版

```
function resolvePromise(promise2, x, resolve, reject) {
	// 1)不能引用同一个对象 可能会造成死循环
	if (promise2 === x) {
		return reject(new TypeError('[TypeError: Chaining cycle detected for promise #<Promise>]----'));
	}
	let called;// promise的实现可能有多个，但都要遵循promise a+规范，我们自己写的这个promise用不上called,但是为了遵循规范才加上这个控制的，因为别人写的promise可能会有多次调用的情况。
	// 2)判断x的类型，如果x是对象或者函数，说明x有可能是一个promise，否则就不可能是promise
	if((typeof x === 'object' && x != null) || typeof x === 'function') {
		// 有可能是promise promise要有then方法
		try {
			// 因为then方法有可能是getter来定义的, 取then时有风险，所以要放在try...catch...中
			// 别人写的promise可能是这样的
			// Object.defineProperty(promise, 'then', {
			// 	get() {
			// 		throw new Error();
			// 	}
			// })
			let then = x.then; 
			if (typeof then === 'function') { // 只能认为他是promise了
				// x.then(()=>{}, ()=>{}); 不要这么写，以防以下写法造成报错， 而且也可以防止多次取值
				// let obj = {
				// 	a: 1,
				// 	get then() {
				// 		if (this.a++ == 2) {
				// 			throw new Error();
				// 		}
				// 		console.log(1);
				// 	}
				// }
				// obj.then;
				// obj.then

				// 如果x是一个promise那么在new的时候executor就立即执行了，就会执行他的resolve，那么数据就会传递到他的then中
				then.call(x, y => {// 当前promise解析出来的结果可能还是一个promise, 直到解析到他是一个普通值
					if (called) return;
					called = true;
					resolvePromise(promise2, y, resolve, reject);// resolve, reject都是promise2的
				}, r => {
					if (called) return;
					called = true;
					reject(r);
				});
			} else {
				// {a: 1, then: 1} 
				resolve(x);
			}
		} catch(e) {// 取then出错了 有可能在错误中又调用了该promise的成功或则失败
			if (called) return;
			called = true;
			reject(e);
		}
	} else {
		resolve(x);
	}
}

```

主要就是多了resolvePromise这么一个函数，用来递归处理then内部回调函数执行后的结果，它有4个参数：

promise2: 就是新生成的promise，这里至于为什么要把promise2传过来后面会介绍。
x: 我们要处理的目标
resolve: promise2的resolve, 执行之后promise2的状态就变为成功了，就可以在它的then方法的成功回调中拿到最终结果。
reject: promise2的reject, 执行之后promise2的状态就变为失败，在它的then方法的失败回调中拿到失败原因。


# Promise的then的第二个参数和catch的区别

* reject是用来抛出异常的，catch是用来处理异常的；
* reject是Promise的方法，而then和catch是Promise实例的方法（Promise.prototype.then 和 Promise.prototype.catch）。

区别 

主要区别就是，如果在then的第一个函数里抛出了异常，后面的catch能捕获到，而then的第二个函数捕获不到。

```
const promise = new Promise((resolve, rejected) => {
    throw new Error('test');
});

//此时只有then的第二个参数可以捕获到错误信息
promise.then(res => {
    //
}, err => {
    console.log(err);
}).catch(err1 => {
    console.log(err1);
});


//此时catch方法可以捕获到错误信息
promise.then(res => {
    //
}).catch(err1 => {
    console.log(err1);
});


//此时只有then的第二个参数可以捕获到Promise内部抛出的错误信息
promise.then(res => {
    throw new Error('hello');
}, err => {
    console.log(err);
}).catch(err1 => {
    console.log(err1);
});

//此时只有then的第二个参数可以捕获到Promise内部抛出的错误信息
promise.then(res => {
    throw new Error('hello');
}, err => {
    console.log(err);
});


//此时catch可以捕获到Promise内部抛出的错误信息
promise.then(res => {
    throw new Error('hello');
}).catch(err1 => {
    console.log(err1);
});
```

两个方法的比较

```
// bad 适用于单个 链式的捕捉不到
promise
  .then(function(data) {
    // success
  }, function(err) {
    // error
  });

// good 
promise
  .then(function(data) { //cb
    // success
  })
  .catch(function(err) {
    // error
  });
```

上面代码中，第二种写法要好于第一种写法，理由是第二种写法可以捕获前面then方法执行中的错误，也更接近同步的写法（try/catch）。因此，建议总是使用catch方法，而不使用then方法的第二个参数。
