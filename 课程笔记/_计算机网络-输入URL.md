[toc]

# URL

`Uniform Resource Locator`统一资源标识符，是浏览器用来检索web上公布的任何资源的机制

组成：
```js
protocol + domain + port + path + params + anchor

http://www.example.com:80/path/to/myfile.html?key=value#SomewhereInTheDocument

protocal: http
domain: www.example.com// 网站名，www服务器名，example.com域名
port: :80// 如果使用标准端口，可以省略
path: /path/to/myfile.html
params: ?key=value
anchor: #SomewhereInTheDocument// 不会发送到请求的服务器
```

# 域名

`domain names`，为互联网上任何可用的网页服务器提供方便人类理解的地址，指向某一个IP地址。**从右到左阅读。**

```js
developer.mozilla.org// label2 + label1 + TLD
www.taobao.com// 服务器名 + SLD + TLD

TLD: top-level domain 顶级域名，如.com商业/.org组织/.net网络/.cn中国等
label: 标签或组件，紧随着TLD，包含字母、0-9、-（仅出现在中间）。一个域名可以有多个标签
SLD: secondary level domain 二级域名，刚好位于TLD前面的标签
主机：www网站/mail邮箱
```


# 浏览器输入URL后执行的全部过程

0. URL解析：判断输入的是一个合法的URL还是一个待搜索的关键词
1. DNS查询：在客户端应用层（首先使用DNS缓存，如果缓存中没有指定的url，就），借助UDP单播，解析服务器域名到客户端
2. 建立TCP连接：在客户端网络层，通过路由器查找路由表确定如何到达服务器（本网络地址-特定主机路由-默认路由-出错）。在客户端链路层，通过ARP协议查找给定IP地址的MAC地址，来定位服务器特定主机位置。三次握手建立TCP连接（**面向有连接的、全双工可靠交付的传输层协议**）。
3. 发送HTTP请求：建立TCP连接后，在此基础上进行通信。请求内容包括请求行、请求头、请求体。
4. 响应请求：服务器逻辑请求完成后，返回一个HTTP响应消息，包括状态行、响应头、响应体。
5. 页面渲染：浏览器对资源进行解析，查看响应头信息，做相应处理（如存储cookie等）

## 浏览器渲染过程

主线程：

1. 构建dom树
2. 构建css树
3. 构建render树（渲染树）
4. 节点布局
5. 页面绘制

渲染线程：

与主线程相互独立，进度不影响主线程。

在节点布局时依据层级关系生成层树，光栅化图帧后将图层分块，优先绘制视口周围区域。（css动画比js高效的原因：分层与合成）

使用translate调用浏览器硬件加速，实际上是创建了一个被传递到GPU处理的层。使用`css will-change: transform | etc.;`属性能够提前通知浏览器动画操作与类型，令其提前优化。注意不能滥用。

### dom树

`(Document Object Model)`。它是HTML文档的对象表示，揭示了DOM对象之间的层次关系，也是外部内容（如js）与HTML元素之间的接口。根节点是`Document`对象。

在构建DOM树过程中，使用状态机对返回文档的字符串进行解析成为token。token种类有很多，如`begintag、endtag`等。遇到`begintag`进栈，遇到`endtag`出栈，直到栈空。

### css树

css文件下载完成后，对css进行解析，将css字节转化为字符，最后链接到css对象模型的树结构。

### 渲染树

从DOM树的根节点开始遍历每个可见节点，然后对每个可见节点找到适配的css样式规则并应用。渲染树生成后，还没办法渲染到屏幕上。因为渲染到屏幕上得到各个节点的位置信息，需要布局处理。

### 节点布局

由于渲染树的每个节点都是一个`render object`对象，包含宽高等信息。所以浏览器可以通过这些样式信息确定每个节点对象在页面上的确切大小和位置，即输出盒子模型。

### 绘制

遍历渲染树，组合指令成为列表，调用渲染器`paint()`方法在屏幕上显示内容。渲染树的绘制工作由浏览器的UI后端组件完成。

### 其他

1. dom树与css树并行加载。

2. 渲染阻塞：在渲染过程中遇见js标签的时候，先去执行js内容，直至脚本完成执行。之后继续构建DOM。

## `<script>`defer与async

### `<script>`
浏览器在解析`html`的时候，如果遇到一个没有任何属性`script`标签，就会暂停解析，先发送网络请求获取该js脚本的代码内容，然后让js引擎执行该代码，当代码执行完毕后恢复解析

![img](https://pic1.zhimg.com/80/v2-566e1583691ad6fdf6479bd21a2d549c_720w.png)

### `<script async>`
浏览器在解析`html`的时候，遇到带`async`属性的`script`标签，标志该脚本的网络请求是异步的，不会阻塞浏览器解析`html`。**若网络请求回来后，`html`还没有解析完，浏览器会暂停解析，令js优先执行。**（`async`是不可控的，执行时间不确定。如果存在多个`async`，执行顺序也不确定，完全依赖网络传输结果）。因此适用加载互不依赖的库。

![img](https://pic3.zhimg.com/80/v2-74febba4b5c4b370a7a7edea01cb5cd2_720w.png)

### `<script defer>`
浏览器在解析`html`的时候，遇到带`defer`属性的`script`标签，标志该脚本的网络请求是异步的，不会阻塞浏览器解析`html`。**若网络请求回来后，`html`还没有解析完，浏览器不会暂停解析，令js等待`html`解析完毕后再执行js代码。**如果存在多个`defer`，会保证它们按照出现的顺序执行，不会破坏js脚本之间的依赖关系（IE9及以下除外）

![img](https://pic3.zhimg.com/80/v2-e5fef119db5c0039aa3fb6674a5d3026_720w.png)

> 将`<script>`写在`<body>`最后执行：防止加载资源而导致长时间的白屏；js可能进行dom操作