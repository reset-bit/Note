[toc]

# Promise

站在使用者角度，有一个异步耗时的任务要做，任务成功后执行指定函数，任务失败后执行另一个指定函数。

`new Promise()`返回一个`promise`对象，是对异步操作封装的描述。异步操作成功会自动调用`then()`的第一个参数函数，否则调用`then()`的第二个参数函数。

> 约定：
>
> - 在本轮**事件循环**运行完成之前，回调函数不会被调用
> - 即使异步操作已经完成（不论成功或失败），在这之后添加的then()添加的回调函数也会被调用执行
> - 通过多次调用then()可以添加多个回调函数，会按照插入顺序执行 => **链式调用**

## `then()`

**`then()`会返回一个和原来不同的新的`promise`对象**：优先返回`then()`内执行函数返回的`promise`对象，否则总返回一个成功的`promise`对象。第二个`then()`的参数为前一个`then()`执行函数的返回值，因此**总是建议显式返回一个`promise`对象。**

> then()返回失败promise对象的情况：
>
> - 执行函数直接返回失败的promise对象（如`Promise.reject()`）
> - 执行函数报错或异常情况。

## `catch()`

`then()`返回失败`promise`对象，可以直接跳转`catch()`。当前一般仅使用`then()`关注成功情况，使用`catch()`关注失败情况，但`catch()`不一定返回失败的`promise`。失败的`promise`必须要有`catch()`处理，否则会报错。

`catch()`只捕捉在其之前，同时是其作用域的异常。

> then(resolve, reject)中reject与catch区别：
>
> - reject是用来抛出异常的，catch是用来处理异常的
> - reject是promise的方法，catch是promise实例的方法 (promise.protptype.catch)
> - 三者捕获错误信息时使用就近原则，即若promise内部存在错误，自reject和catch都存在的情况下，只有reject能捕获到错误

## `finally()`

返回一个`promise`，在`promise`结束时，不论结果是`fulfilled/rejected`，都会执行指定的回调函数。由于无法知道`promise`的最终状态，故`finally()`回调函数中不接受任何参数，并且不改变`promise`的状态。

## `all()`

接收一个数组参数，数组内容为异步函数，这些异步函数是并行执行的。**当异步函数都执行完成后**，`promise.all()`会将它们返回的数据放到一个数组中传给`then()`。应用场景如并发加载图片等。

## `race()`

使用方法与`promise.all()`类似，但`promise.race()`是**以最先结束运行的函数作为基准执行回调函数，在这个过程中其他异步函数并不会停止执行**。应用场景如为图片设置超时时间（让图片加载函数与延时函数一同放入`race()`中）等。


```javascript
// 模拟promise
// status,successHandler,failHandler
function Promise(asyncFun) {
    this.status = 'pending';// fulfilled 成功 | rejected 失败
    this.successHandler = null;
    this.failHandler = null;
    function resolve() {
        this.status = 'fulfilled';
        if(this.successHandler) {
            this.successHandler(data);
        }
    };
    function reject() {
        this.status = 'rejected';
        if(this.failHandler) {
            this.failHandler(data);
        } else {
            throw new Error('Uncaught Promise rejected!');
        }
    };
    asyncFun(resolve.bind(this), reject.bind(this));
};
Promise.prototype.then = function(successHandler, failHander) {
    this.successHandler = successHandler;
    this.failHandler = failHander;
};
Promise.prototype.catch = function(failHandler) {
    return this.then(null, failHandler);
};

// resolve 决心、决心要做的事; reject 拒绝、丢弃
new Promise(function(resolve, reject) {
    if(new Date().getSeconds() % 2 === 0) {
        resolve('吃饭');
        reject('睡觉');
    }
}).then(
    function(data) { console.log(data); },
    function(data) { console.log(data); }
);

// 混用旧式回调（使用旧方式来传递成功或者失败的回调）和promise可能会造成运行时序问题
// 地狱回调，可维护性差
setTimeout(() => {
    console.log('go shopping');
    setTimeout(() => {
        console.log('go shopping end');
    }, 1000);
}, 1000);
// 封装问题函数，避免地狱回调
const wait = time => new Promise(resolve => setTimeout(resolve, time));
wait(1000).then(() => {
	console.log('go shopping');
	return wait(1000);// 返回promise对象，仅当成功才被then捕获
}).then({
    console.log('go shopping end');
});
```

# Ajax(Asynchronous Javascript and xml)

异步`Javascirpt和XML`，核心是js对象`XMLHttpRequest`。

完成`HTTP`请求需要：

1. 请求网址、请求方法`GET/POST`
2. 提交请求内容、请求主体等
3. 接收相应内容

## 发送`Ajax`请求的5个步骤

1. 创建异步对象，即`XMLHttpRequest`对象
2. 设置请求参数，包括：请求方法、请求url、是否异步
3. 发送请求
4. 注册事件：`onreadystatechange`事件，状态改变时调用
5. 获取返回数据

## 自封装ajax

```javascript
document.querySelector('#btn-ajax').onclick = function() {
    // get
    var ajaxObjGet = new XMLHttpRequest();
    ajaxObjGet.open('get', 'ajax-get.php');// method,url,async
    ajaxObjGet.send();
    // 每当readyState属性改变时，就会调用onreadystatschange函数
    /**
    	readyState:
    	0 请求未初始化
    	1 服务器连接已建立
    	2 请求已接收
    	3 请求处理中
    	4 请求已完成，响应已就绪
    	
    	status:
    	200 ok
    	404 未找到页面
    */
    ajaxObjGet.onreadystatechange = function() {
        if(ajaxObjGet.readyState === 4 && ajaxObjGet.status === 200) {
        	console.log('数据返回成功');
        }
    }
    // post
    var ajaxObjPost = new XMLHttpRequest();
    ajaxObjPost.open('post', 'ajax-post.php');
    ajaxObjPost.setRequestHeader('content-type', 'application/x-www-form-urlencoded');// 使用post提交数据，添加http头，设置请求头、编码方式
    ajaxObjPost.send('name=fox&age=18');// 添加要发送的数据
    ajaxObjPost.onreadystatechange = function() {
        if(ajaxObjPost.readyState == 4 && ajaxObjPost.status == 200) {
           console.log(ajaxObjPost.responseText);// 获得字符串形式的响应数据
           console.log(ajaxObjPost.responseXML);// 获取XML形式的响应数据
        }
    };
};

// jquery.ajax
$.ajax({
    type: 'post',
    url: url,
    dataType: 'html',
    async: true,// 异步请求
    success: function() {},
    complete: function(XMLHttpRequest, textStatus) {},// 请求完成时的调用函数
    error: function() {}
});
```

