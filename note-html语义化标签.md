# 语义化标签

HTML5中新增语义化标签主要有：`<header>/<hgroup>/<nav>/<aside>/<article>/<section>/<footer>/<address>/<strong>`等

优点：

- 比`<div>`标签有更加丰富的含义，方便开发与维护
- 利于搜索引擎优化SEO
- 方便其他设备解析（移动设备、盲人阅读器等）
- 利于合作，遵守W3C标准

使用规范：

- 尽可能少用无语义标签`<div>/<span>`
- 在语义不明显、既可以用`<div>`也可以用`<p>`时，尽量用`<p>`，因为`<p>`在默认情况下有上下间距，对兼容特殊终端有利
- 表格用`caption/th/td`；表单`input`用`label`

### header与hgroup

一般用于放置导航栏或标题。

一个页面中可以包含多对`<header>`，可以为任何需要添加标题的区块标签添加`<header>`元素，如`<article>/<section>`。

如果有连续多个h1-h6标签就用`<hgroup>`。

### nav

表示页面的导航。一个页面中可以包含多对`<nav>`，可以在`<header>/<aside>`等标签内使用。

为了搜索引擎解析，最好是将主要的链接放在`<nav>`中。

### aside

包含的内容不是页面的主要内容，是对页面的补充。一般用于页面、文章的侧边栏、广告、友情链接等区域。

### article

使用在相对比较独立、完整的内容区块，如论坛帖子、新闻报道、用户评论等。

### section

一组或一节内容。

> div：应用广泛，任意一个区域
>
> article：需要一个单独的模块来实现一个单独的功能
>
> section：包含的内容是一个明确的主题，通常有标题区域

### footer

一般被放置在页面或页面中某个区块的底部，包含版权信息、联系方式等。一个页面可以有多个`<footer>`。

### address

作为联系信息、邮编地址、邮件地址等出现，区块容器。

### strong

强调标签

```html
<header>this is header</header>
<nav>
    <li><a href="">this is a-1</a></li>
    <li><a href="">this is a-2</a></li>
</nav>
<article>
    this is article
    <section>
        this is section
    </section>
</article>
<aside>this is aside</aside>
<footer>
    this is footer
    <strong>strong</strong>
    <address>this is address</address>
</footer>
```

