# jQuery
优点：写的更少，做得更多；兼容性好

原生js中dom操作的许多属性，在jQuery中都变成了函数调用

jQuery中很多方法返回自身，所以支持链式操作

```javascript
// 原生js dom->jQuery dom
var box = document.querySelctor('.box');
$(box).text();
// jQuery dom->原生js dom
$('.box')[pos].innerText
$('.box').get(pos).innerText

// 获取dom对象
// 原生js
document.getElementByID();
xx.getElementByTagName();
xx.getElementsByName();
xx.querySelector();
xx.querySelectorAll();
// ------------up
xx.parentNode
// ------------down
xx.children
// jQuery
$();//返回值一定是一个jQuery对象集合[object Object]，是否找到元素的区别在于返回值length是否为0
xx.each();// 规定每个元素运行的函数，return false可提前停止循环
// ------------up
xx.parent(); 
// 以下方法均支持arg，为css选择器字符串
xx.parents();
xx.prev(); xx.next();// 前面/后面紧挨着的兄弟元素
xx.prevAll(); xx.nextAll();// 前面/后面所有兄弟元素
xx.siblings();// 除自己以外的所有兄弟元素
xx.closest();// 祖先中距离最近的指定元素
// ------------down
xx.children();
xx.find();// 后代中满足要求的元素
xx.filter();// 对已经找到的结果，进行进一步的过滤筛选
xx.not();// 从匹配元素集合中删除元素
xx.index();// 当前元素在兄弟中从0开始计数的下标位置。无参数。
xx.eq();// 匹配指定索引值的元素，从0开始

// 取设双标记dom对象文本内容
// 原生js
xx.innerText
// jQuery
xx.text([args]);

// 取设双标记dom对象html内容
// 原生js
xx.innerHTML
// jQuery
xx.html([args]);

// class值相关
// 原生js
xx.className
xx.classList.add();
xx.classList.remove();
xx.classList.toggle();
xx.classList.contains();
// jQuery
xx.addClass('arg1 arg2 ...');
xx.removeClass();
xx.toggleClass();// 支持定向切换与批量切换，多类名以空格分隔
xx.hasClass();

// 设置style值
// 原生js
xx.style.name = value;
// jQuery
xx.css(name, value);// 单个样式控制
xx.css({ name: value, name: value });

// 取设属性值
// 原生js
img.src// 专用属性
xx.getAttribute(name); xx.setAttribute(name, value);// 自定义属性
xx.dataset.name // <a data-id=1></a>，取出来仍然是字符串
xx.value
// jQuery
xx.attr(name); xx.attr(name, value);
xx.prop(name); xx.prop(name, value);// 返回true/false
// 如checkbox，attr('checked'):checked; prop('checked'):false;
// 只添加属性名该属性就会生效；只存在true/false值的属性应当使用prop()
xx.val(); // 表单元素vlaue

// 创建、删除、添加、克隆标签
// 原生js
document.createElement(name);
parent.removeChild(child);
// 清空双标记标签内容通过设置innerHTML来控制
parent.appendChild(child);
parent.insertBefore(child, reference);
// jQuery
$('htmlstr')// new htmlstr
target.remove();
xx.empty();// 清空双标记标签中所有内容
parent.append(child); child.appendTo(parent);
parent.prepend(child); child.prependTo(parent);// 向头部追加元素
reference.before(newNode); newNode.insertBefore(reference);// 向指定元素之前添加元素
reference.after(newNode); newNode.insertAfter(reference);
xx.clone([true | false]);// true表示将节点连同事件一起克隆，默认为false
oldNode.replaceWith(newNode); newNode.replaceAll(oldNode[s]);
target.wrap(wrapper); wrapper.wrapInner(target);
```

```java
// 事件绑定，隐式迭代，不会被替代
xx.click(function() {
    console.log($(this).index());// this仍指向js dom对象
});
xx.on("click", ".btn-update", function(e) {// e在js事件对象之上做了丰富
    console.log(e.delegateTarget);
});
xx.trigger();// 事件模拟
```

