# vue渲染过程

`template -> ast -> render -> virtual dom -> ui`

1. 模板解析生成AST树（AST是**Abstract Syntax Tree**抽象语法树的简称，是源代码的抽象语法结构的树状表现形式）
2. AST树解析生成对应的render渲染函数
3. 数据通过render函数解析生成VNode（虚拟节点）
4. VNode经过diff算法处理后，生成真实dom展示

## render()

vue核心技术使用了虚拟dom，在项目中template模板需要解析编译成虚拟dom，`render()`即在此过程中发挥作用。

使用`render()`描述虚拟dom时，借助`createElement()/简写为h()`来作为构建虚拟dom的工具。

```js
// style 1
// createElement3个参数：HTML标签字符串，包含模板相关属性的字符串，子节点（由createElement()构建而成）或字符串文本节点
new Vue({
    el: '#app',
    render: function(createElement) {
        return createElement('h2', {class:'box'}, ['hello world']);
    }
});
// style 2
new Vue({
    el: '#app',
    render: function(createElement) {
        return createElement(App)
    }
});
// or
new Vue({
    el: '#app',
    render: h => h(App)
});
```

## diff算法

diff算法主要将新旧dom树进行对比，找出之间的差异，来渲染网页。

diff算法只比较同层节点，时间复杂度`o(n)`：

- 数据改变时，触发setter，通过dep通知所有订阅者。订阅者调用`patch()`比较Vnode和OldVnode。如果不一样则`return Vnode`，将Vnode真实化后替换dom节点
- 如果Vnode和OldVnode值得进一步比较，则调用`patchVnode()`方法。
  - Vnode有子节点，OldVnode没有：将Vnode子节点真实化后添加到真实dom上
  - Vnode没有子节点，OldVnode有：将真实dom上相应节点删掉
  - 二者都有文本节点但内容不同：将真实dom上文本内容替换为Vnode上文本内容
  - 二者都有子节点：调用`updateChildren()`进一步比较。`updateChilren()`使用首尾指针法对比，新的子节点集合和旧的子节点集合，各有首尾2个指针

# 虚拟Dom

本质是一个普通的js对象，包含`tag/props/children`三个属性。

**虚拟dom算法操作真实dom，性能高于直接操作真实dom。**虚拟dom算法=虚拟dom+diff算法。

优势：

- 使用diff算法对比js原生对象，计算出需要变更的dom，只对变化的dom进行操作，减少操作真实dom带来的性能消耗
- 抽象了原本的渲染过程，实现了跨平台的能力（不仅局限于浏览器dom，可以是安卓和ios的原生组件）

```jsx
<div id='app'>
    <p class='text'>hello world!</p>
</div>
// 转换为虚拟dom对象：
{
    tag: 'div',
    props: {
        id: 'app'
    },
    children: [
        {
            tag: 'p',
            props: {
                className: 'text'
            },
            children: [
                'hello world!'
            ]
        }
    ]
}
```

