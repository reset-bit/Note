[toc]

# 执行时机

顺序执行异步操作：“同步优先，异步靠边，回调垫底”

```js
setTimeout(function() {
	console.log(1);
}, 0);
async function async1() {
	console.log(2);
	const data = await async2();
	console.log(3);
	return data;
}
async function async2() {
	return new Promise((resolve) => {
		console.log(4);
		resolve('async2的结果');
	}).then((data) => {
		console.log(5);
		return data;
	});
}
async1().then((data) => {
	console.log(6);
	console.log(data);
});
new Promise(function(resolve) {
	console.log(7);
}).then(function() {
	console.log(8);
});
// 2 4 7 5 3 6 async2
```

# 返回值

```js
// 无返回值
Promise.resolve().then(res => {
    
}).then(data => {
	console. 1og(data):// undefined
});
//返回promise
Promise.resolve().then(res => {
	return 'hello';
}).then(data => {
	console.log(data);// hello
});
// 返回promise
Promise.resolve().then(res => {
	return new promise((resolve, reject) => {
		resolve('str');
    });
}).then(data => {
	console.1og(data):// str
	return promise.resolve();
});
```

# catch&finally
```js
Promise.resolve('11').then((res, resolve, relect) => {
	res === '' && reject();
}).then(() => {
	console.log('this is then');
}).catch(() => {
	console.log('this is catch');
});
// this is then

Promise.resolve('').then((res, resolve, relect) => {
	res === '' && reject();
}).then(() => {
	console.log('this is then');
}).catch(() => {
	console.log('this is catch');
});
// this is catch

Promise.resolve('11').then((res, resolve, relect) =>
	res === '' && reject();
}).then(() => {
	console.log('this is then');
}).catch(() => {
	console.log('this is catch');
}).finally(() => {
    console.log('this is finally');
});
// this is then
// this is finally
```