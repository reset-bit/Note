[toc]

# 生成器与迭代器

```js
function wait(time) {
    new Promise(resolve => {
        setTimeout(resolve, time);
    });
};
// 生成器函数，使用*声明，yield + iterator.next()可令生成器函数断点式执行
// 整个yield表达式的值为undefined
function* generator() {
    console.log('去买菜');
    yield wait(100);// 1
    console.log('买完了，开始做饭');
    yield wait(100);// 2
    console.log('做好了，开始吃饭');// 3
};
// 迭代器函数，对象内含value属性和done属性
// 前者为当前yield暂停处返回值，后者标识generator是否执行到return关键字
let iterator = generator();
iterator.next();// 在1处停止，1开始执行
iterator.next(value);// 在2处停止，并指定前一个（1处）yield表达式返回值由undefined变为value
iterator.next();// 在3处停止，触发return
/**
	ps:由于没有判断返回值中promise的完成状态，因此实际生成器函数中的代码仍然没有阻塞执行：
	{ value: Promise {<pending>}, done: false }
	{ value: Promise {<pending>}, done: false }
	{ value: undefined, done: true }
*/

// 模拟co，令生成器代码阻塞执行
function co(generator) {
    let iterator = generator();
    function genNext(iterator) {
        let { value, done } = iterator.next();
        if(done) return;
        value.then(() => genNext(iterator));
    };
    genNext(iterator);
}
co(generator);
```



# async & await(ES7)

本质上是`promise`语法糖。

使用`async`关键字放在函数声明之前，使其成为`async function`（**异步函数特征之一是保证函数的返回值为`promise`**，若显式return数据，则实际返回一个成功的promise对象，包含指定的数据。使用await可获取promise中注入的值）。

`await`只在异步函数里面才起作用，它会暂停代码在该行上，直到`promise`完成，然后返回结果值。若结果失败，则抛出异常。所以总建议使用`try...catch`块包裹代码块。在暂停的同时，其他正在等待执行的代码就有机会执行了。其后的代码放在**微任务队列**中。若`await`后跟非异步任务（**实际上是将任务封装在`promise`中**），下方代码仍放入微任务队列中。

可以在类/对象方法前面添加`async`，以便让他们返回`promise`。

缺点：代码可能因为大量的`await`而变慢；必须将等待执行的`promise`封装在异步函数中。

```js
async function myFetch() {
    try {
        let response = await fetch('coffee.jpg');
    	let myBlob = await response.blob();
    } catch(err) {
        console.log(err.message);
    }
    
    let objectURL = URL.createObjectURL(myBlob);
    let image = document.createElement('img');
    image.src = objectURL;
    document.body.appendChild(image);
};

// 异步代码顺序执行，注意一定要执行resolve()
async function test() {
    for(let i = 0; i < 3; ++i) {
        await checkGiteeStatus();
        await waitMinutes();
        console.log(i);
    }
}
function waitMinutes() {
    return new Promise(function (resolve, reject) {
        setTimeout(() => {
            console.log('等2秒');
            resolve('等完了');
        }, 2000);
    });
}
function chectGiteeStatus() {
    return new Promise(function(resolve, reject) {
        $.ajax({
                type: "GET",
                url: "https://gitee.com/api/v5/emojis",
                success: function (data, status) {
                    if (status == "success") {
                        console.log("跟gitee通讯正常！");
                        resolve("通讯结束！")
                    }
                },
                error: function (xhr, errorText, errorType) {}
            })
    });
}
/**
	输出：
        跟gitee通讯正常！
        等2秒
        0
        跟gitee通讯正常！
        等2秒
        1
        跟gitee通讯正常！
        等2秒
        2
*/ 
```

