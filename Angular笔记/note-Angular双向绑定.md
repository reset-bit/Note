# 脏数据检查-digest

digest是angular实现**双向数据绑定**的基础

angular需要检查每个scope变量的变化，该过程由scope.\$digest()完成。该方法通常不需要自己调用，angular在每一轮js执行完成后会自动调用**每个scope的digest方法**（包括响应angular内置的directive事件，如ng-click,ng-change等；controller初始化；service回调函数，如\$timeout、\$http等）

digest即脏数据检查，检查所有监视数据是否有改动，需要为每一个变量设置一个监视器（监视函数：获取当前变量最新值-{{}}；监听函数：数据改动时回调-ng-click etc.）。脏数据检查认为**数据发生改变的条件**是当前数据和保存的历史数据不一致，**引用类型数据必须要在原有数据上做改动**

多数情况下angualr能够知晓当前scope变量发生变化，但当执行函数没有通过angular提供的接口（如
setTimeout、ajax回调函数等）改变了数据时，需要手动触发digest，通常调用 \$apply，该函数的特点是传入函数立即执行。也可以通过 \$timeout 服务来触发脏值检查，因为该服务内部调用了\$apply

```js
setTimeout(() => $scope.$apply(() => scope.name = 'jack')});
```

> angular一次只允许一个digest过程执行。如果调用时正在digest，则会抛出异常。
>
> 如何判断调用\$apply时是否正在digest?
>
> 

除\$apply、$timeout之外，还有以下关于digest的函数:

- \$evalAsync：当前scope改动已经有一个digest正在进行，因此不能调用$apply。该方法将需要执行的函数延迟到【当前digest的下一次循环迭代】中执行。===>vue2 时间片 微任务 nextTick
- \$applyAsync：当前scope改动时，频繁执行某些回调圣数（如第三方http服务）。该方法将指定函数放在下一个【degist循环执行】

---

双向数据绑定：

单向数据流也可以有双向数据绑定，vue/react在父子组件间教据传递仍然是单向数据流

- angular：scope.$digest脏值检查，为每一个变量设置监听器（监听函数+监视函数）进行数据监听，在循环检查中更新数据
- vue：数据劫持以达成发布订阅者模式，当DOM数据改变时触发事件并同步
- react：setState+onChange自行实现（react目标是让代码结构更加清晰易于维护，双向绑定实际在一定程度上限制了自由度：如输入validate以及其他数据转换）
