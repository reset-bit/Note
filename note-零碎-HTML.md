[toc]
# 标签综合
## 标签高度
html标签为根节点，高度100%参照为浏览器可视区域。页面没有内容时，复位后html标签高度为0 ；
为body设置背景色，效果总应用到浏览器全部可视区域，不论body自身宽高

## 标签嵌套
a标签嵌套易出现莫名的兄弟标签，需要避开

## 标签属性
所有html标签支持自定义属性，建议命名`data-xxx:xx;`，读取使用`xxx.dataset.xxx`

## 标签内部文本

1. `document.body.innerHTML`：返回HTML化的文本内容，只能用于元素节点
2. `document.getElementById('content').childNode[0].nodeValue`：元素节点返回null，属性节点返回属性值，文本节点返回文本值
3. `document.body.textContent`：返回去HTML化的文本内容
4. `document.body.innerText`：同`textContent`，但对空白字符的解析不同

> 元素节点：HTML标签元素
>
> 属性节点：一般是元素节点的属性
>
> 文本节点：DOM中用于呈现文本的部分，一般包含在元素节点的内部

## 置换元素与不可置换元素
置换元素：浏览器根据元素的标签和属性，来决定元素的具体显示内容。如`<img>/<input>/<select>/<textarea>`
不可置换元素：内容直接表现给客户端。如`<label>`

# 浏览器相关
## 浏览器排版
目前有3种模式：怪异模式、接近标准模式、标准模式。怪异模式是为了不弃用原先的标准，模拟IE等的非标准行为。在接近标准模式下，有少数的怪异行为实现。
对HTML文件来说，浏览器使用文件开头的`DOCTYPE`决定使用怪异模式或标准模式处理。对于没有文档类型声明或者文档类型声明不正确的文档，浏览器就会使用怪异模式解析和渲染该文档。
标准模式与怪异模式的常见区别：

- 标准模式盒模型宽高=内容区宽高；怪异模式盒模型宽高=内容区宽高+内边距padding+边框border
- 标准模式行内元素基线对齐；怪异模式行内元素下边框对齐


# 标签实例
## DL DT DD
适用于商品列表、页脚等，块级元素
```
dl definition list 自定义列表
dt definition term 自定义项目
dd definition description 自定义描述
<dl>
	<dt></dt>
	<dd></dd>
	<dt></dt>
	<dd></dd>
</dl>
```

## form表单

### input

`<input type='password'>`内容总为`string`类型

### disabled & readonly
共同点：设为true
不同点：disabled该表单输入项不能获取焦点，用户所有操作无效，提交表单时该项不会被提交；readonly只针对文本输入框，用户只是不能编辑对应文本，提交表单时该项会被提交

### H5新增表单属性
- keygen提供一种验证用户的可靠方法，生成一对公钥和私钥。私钥存放在浏览器本地，公钥存放在服务器。优点是提高验证安全性，使用TLS/SSL安全传输协议；缺点如IE/Safari不支持、用户交互差等
- output用于不同类型的输出，会随for属性绑定的表单内容变化而变化（需要另外为form绑定oninput等事件）

- datalist子项option，规定输入域选项列表（类似联想提示）
- optgroup子项option，父级select，作子项标题
- fieldset将表单内相关元素分组；legend为filedset元素定义标题
```html
<form action="" oninput="x.value=parseInt(a.value)+parseInt(b.value)">
    <fieldset id="">
        <legend>内容标题</legend>

        主修课程:<input type="text" list="course">
        <datalist id="course">
            <option value="计算机组成原理"></option>
            <option value="软件工程"></option>
        </datalist>

        <select>
            <optgroup label="label1">
                <option value ="">value1.1</option>
                <option value ="">value1.2</option>
            </optgroup>
            <optgroup label="label2">
                <option value ="">value2.1</option>
            </optgroup>
        </select>

        <input type="number" id="a" value="0"> + <input type="number" id="b" value="0">
        = <output for="a b" name="x">0</output>
	</fieldset>
</form>
```
- a标签href属性`mailto:cuidongping22@163.com`可在网页上直接打开邮件客户端发送邮件

## href与src

`href`表示超文本引用，用在`link/a`等元素上，在当前元素和引用资源之间建立联系。

`src`表示引用资源，用在`img/script/iframe`上，指向的内容会迁入到文档中当前标签所在的位置。

当浏览器解析到`<script src="xxx.js"></script>`时，会**暂停其他资源的下载和处理，直至将该资源加载、编译、执行完毕**。当浏览器解析到`<link href="xxx.css" rel="stylesheet"></link>`时，**下载对应css文件，但不停止对当前文档的处理**。

## embed

将外部内容嵌入文档中的指定位置，此内容由外部应用程序或其他交互式内容源（如浏览器插件）提供。

属性：`height/width/src（被嵌套资源url）/type（MIME类型）`

> 大多数现代浏览器已经弃用并取消了对浏览器插件的支持，所以如果您希望您的网站可以在普通用户的浏览器上运行，那么依靠 `<embed>` 通常是不明智的。

```html
<embed src='hello.swf' />
```



## figure

规定独立的流内容（图像、图标、照片等），内容应与主内容相关。如果被删除，不应对文档流产生影响

```html
<figure>
	<figcaption>黄浦江上的卢浦大桥</figcaption>
    <img src-'...'/>
</figure>
```

