---
title: axios-cancelToken 总结
date: 2019-10-08 17:13:45
categories: 
- web前端
tags: vue
---

在vue 项目中我们通常会使用axios 来请求接口。在项目中遇到列表切换，防止快速多次点击列表导致请求频繁发送等等减轻服务器压力cancelToken取消请求就是一方案。 在ajax时代是用的abort()来取消接口请求的。 当然也有其他方法比如：节流、按钮置灰等等

### 基本用法
看下官网的基本用法
[cancelToken 取消请求] (http://www.axios-js.com/zh-cn/docs/#%E5%8F%96%E6%B6%88)

#### 官网方法一

```
var CancelToken = axios.CancelToken;
var source = CancelToken.source();

axios.get('/user/12345', {
  cancelToken: source.token  // 标记
}).catch(function(thrown) {
  if (axios.isCancel(thrown)) { // 主动取消的在这可以捕捉到
  } else {
    // 处理错误
  }
});

// 取消请求（message 参数是可选的）
source.cancel('Operation canceled by the user.');

// post 取消请求
axios.post('/user/123/',{
    name:x
},{
    cancelToken:source.token // 是在第三个地方添加 
}).then(res => {
    // 成功
}).catch(err => {
    if(axios.isCancel(err)){
        // 主动取消
    }else {
        //其他错误
    }
})
```

#### 官网方法 二 可以通过传递一个 executor 函数到 CancelToken 的构造函数来创建 cancel token：

```
const ConcelToken = axios.CancelToken
let cancel
axios.get('/user/12345', {
  cancelToken: new CancelToken(function executor(c) {
    // executor 函数接收一个 cancel 函数作为参数
    cancel = c;
  })
});

// cancel the request
cancel();
```

##看下在项目中如何使用

### 防止重复请求
在做项目时会遇到用户连点按钮的情况造成服务器压力或是未知异常等， 为了避免这种情况可以给用户一个友好的提示，多次请求只有最后一次有效

```
     sendRequest() {
            let that = this;
            if(this.cancelList){
                this.cancelList.cancel('我取消啦')
            }
            this.cancelList = axios.CancelToken.source();
            axios.get('http://jsonplaceholder.typicode.com/comments', {
                cancelToken:that.cancelList.token
                })
            .then(res => {
                // 你的逻辑
                this.list = res
            })
            .catch(thrown => {
                if (axios.isCancel(thrown)) {  // 如果调用了cancel方法，那么这里的res就是cancel传入的信息
                    alert('请不要点击过于频繁')
                } else {
                    // 处理错误
                    alert('接收400 401 等其他错误')
                    console.log('处理错误', thrown.message);
                }
            })
        },
```
我们可以把他封装起来
```
const clearHttpPendingList = (config) => {
	Vue.prototype.pending.forEach((item, index) => {
		if (config.url === item.url) {
			item.c();
			Vue.prototype.pending.splice(index, 1);
		}
		if (config.url.includes('http://jsonplaceholder.typicode.com/comments') && localStorage.getItem('comments')) {
			// 不同页面相同的接口
			Vue.prototype.pending.splice(index, 1);
		}
	});
};
axios.interceptors.request.use(
	(config) => {
		clearHttpPendingList(config);
		config.cancelToken = new cancelToken(function executor(c) {
			Vue.prototype.pending.push({ url: config.url, c: c });
		});
		// 发送请求之前做的
		return config;
	},
	(err) => {
		// 请求错误
		return Promise.reject(err);
	}
);
```
> 连续点后它的网络请求
> ![微信截图_20191009151456.png](http://ww1.sinaimg.cn/large/006xVFBigy1g7rz0p0rhfj30n50cu3yu.jpg)


### 取消上一个页面的pedding请求

优化性能， 跳转之前取消 正在请求的接口
```
Vue.prototype.pending = []; // //声明一个数组用于存储每个ajax请求的取消函数和ajax标识 请求的缓存

axios.interceptors.request.use(
	(config) => {
		clearHttpPendingList(config);
		config.cancelToken = new cancelToken(function executor(c) {
			Vue.prototype.pending.push({ url: config.url, c: c });
		});
		// 发送请求之前做的
		return config;
	},
	(err) => {
		// 请求错误
		return Promise.reject(err);
	}
);

router.afterEach((to, from) => {
	Vue.prototype.pending.forEach((item) => {
		item.c();
	});
	Vue.prototype.pending = [];
});
```
这个接口处于pedding状态
![微信图片_20191016135317.png](http://ww1.sinaimg.cn/large/006xVFBigy1g80005p77aj30md01dglg.jpg)
跳转到其他页面取消
![微信截图_20191016135422.png](http://ww1.sinaimg.cn/large/006xVFBigy1g8001as3vaj30hj0510sv.jpg)

### 相同的接口不再调用的应用-
项目中有的接口在多个页面调用， 这个接口返回的数据不会轻易改变。 这个时候我们就会用到这个canceltoken和页面缓存。比如这个重复接口是A， 思路是 先调用A接口然后把接口内容存到本地， 这样在任何页面调用A接口的时候就直接取消掉了， 

看下代码
```
axios.interceptors.request.use(
	(config) => {
		clearHttpPendingList(config);
		config.cancelToken = new cancelToken(function executor(c) {
			Vue.prototype.pending.push({ url: config.url, c: c });
		});
		// 发送请求之前做的
		return config;
	},
	(err) => {
		// 请求错误
		return Promise.reject(err);
	}
);

axios.interceptors.response.use(
	(response) => {
		if (response.config.url.includes('http://jsonplaceholder.typicode.com/comments')) {
			localStorage.setItem('comments', JSON.stringify(response.data));
		} // comments 要缓存的接口
		clearHttpPendingList(response.config); //在一个ajax响应后再执行一下取消操作，把已经完成的请求从pending中移除

		// 响应成功的数据
		return response;
	}
)

const clearHttpPendingList = (config) => {
	Vue.prototype.pending.forEach((item, index) => {
		
		if (config.url.includes('http://jsonplaceholder.typicode.com/comments') && localStorage.getItem('comments')) {
			// 不同页面相同的接口
            item.c();
			Vue.prototype.pending.splice(index, 1);
		}
	});
};
```
