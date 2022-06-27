[toc]

# 跨页面数据共享

以字符串形式存放；使用JSON存取数据；存放到客户端浏览器，数据可被客户读取和篡改，不能存保密性信息
## cookie
最多可以存4KB，一个站点最多保存20个cookie；分为session/会话cookie（会话结束消失，存在浏览器进程内存里）、持久性cookie（存在硬盘里）

通常由服务器端在请求头中设置`setCookie`来开启cookie；服务器端先发送一个cookie对象，该对象会跟随http请求在客户端和服务器来回传递，由浏览器保存在客户端电脑上；服务器端可设置cookie只读（`HttpOnly`属性）；涉及到指定格式

### 内容

name；value；0个或多个属性（键值对），存储cookie的有效期、域、标志等。

### 作用

session管理、个性化（记录用户信息用于下次展示）、追踪（分析日志中关于cookie的记录，搜集用户的购买习惯等信息）

### 跨域

主域可以设置或删除子域的cookie，子域无法设置或删除主域的cookie。

cookie中，domain表示cookie所在的域，默认为请求的地址。跨域则将服务器端 cookie domain字段设置相应域名即可

> 域名/域`Domain Name/Domain`：用作识别网站。几级域名就看几个点
>
> 一级域名：`baidu.com`
> 二级域名：`play.baidu.com`
> 子域名：是一个另外的网页，有独立不同的内容，但不是一个新的域名。`en.dataplugs.com`是`dataplugins.com`的子域名
>
> 如域A为love.haorooms.com，域B为resource.haorooms.com，
> 那么在域A生产一个令域A和域B都能访问的cookie就要将该cookie的domain设置为.haorooms.com；
> 如果要在域A生产一个令域A不能访问而域B能访问的cookie就要将该cookie的domain设置为resource.haorooms.com。

## session

关闭浏览器服务器端session对象并不会被马上清除。因为服务器端并不关心客户端浏览器是否被关闭，它有自己的一条机制来管理session，如多长时间清除没有使用过的session对象等

```js
// 创建或获取专属于当前会话的session对象
HttpSession session = request.getSession(true);// true表示没有session对象自动创建新的；false表示没有返回null
// 存取
session.setAttribute(String name, Object obj);
session.getAttribute(String name);
session.removeAttribute(String name);
```

> cookie与session联系与区别：
>
> 1. cookie在客户端记录信息确定用户身份，不占用服务器端资源，包含一个唯一的session id；session通过在服务端记录信息确定用户身份，客户端禁用cookie则session失效
>
> 2. cookie存储限制见上文；session存储没有上限，但出于服务器端性能考虑，session要设置删除机制
> 3. cookie只能保管ASCII字符串，需要通过编码存储为unicode字符或二进制数据；session能存任何类型的数据
> 4. 可以设置cookie属性，使其达到长期有效的效果；session依赖名为JSESSIONID的cookie，默认为-1，不能达到长期有效的存储
> 5. cookie对并发用户友好，session会耗费大量内存
> 6. cookie支持跨域访问；session不支持跨域

## sessionStorage

最多可以存5MB；默认浏览器关闭/会话结束即消失，无法改变生命周期
```
sessionStorage.setItem(key, value);
sessionStorage.getItem(key);// 无法取得返回null
sessionStorage.removeItem(key);
sessionStorage.clear();
```

## localStorage
最多可以存5MB；除非显式清除（代码手动清除、清除浏览器缓存等），否则一直都在
```
localStorage.setItem(key, value);
localStorage.getItem(key);
localStorage.removeItem(key);
localStorage.clear();
```

## JSON
公认的数据交换形式
```
	JSON.stringify()// 返回字符串
	JSON.parse()// 返回原类型
````

# 浏览器缓存

`cookie`是`http headers`的一个字段，`cookie + session`是为了确保通信过程的有状态性（`http`是无状态协议，无法判断用户请求之间的区别）

`localStorage`和`cookie`类似，记录用户的一些行为和配置。

缓存能够减少浏览器请求资源的次数，减少网络延迟，加快页面响应速度，增强用户体验，减少服务器压力。

## memory cache、disk cache

`memory cache`：内存缓存，暂时性缓存。一旦关闭浏览器，用于缓存的内存空间也会被释放掉。受限于计算机内存大小，仍然需要硬盘来存储大量缓存。

`disk cache`：硬盘缓存，相比于内存缓存优势是长时效。根据`http headers`中的字符段类型，来判断资源是否需要重新请求。如果当前内存使用率高的话，请求资源大概率会被缓存到`disk cache`。

## 分类

**对频繁变动的资源**，使用`Cache-Control:no-cache`使浏览器每次都请求服务器。虽然不能节省请求数量，但能显著减少相应数据大小。

**对不常变化的资源**，使用`Cache-Control:max-age=31536000`配置一个很大的有效时间，令浏览器请求相同的URL会命中强缓存。

**HTML文件**不能设置强缓存。一旦不能更新，是灾难级的技术故障。

**CSS/JS/img等资源**，可以设置一个长时间的强缓存。一旦文件发生改变，就令浏览器请求新的资源。

### 强缓存

不会向服务器发送请求，直接从缓存中读取资源。设置`http header`中`Expires/Cache-Control`来实现。

对`Cache-Control:max-age=300`，表示在请求正确返回时间5分钟内再次加载资源，就会命中强缓存。

### 协商缓存

强制缓存失效后，浏览器携带缓存标识向服务器发起请求，由服务器根据缓存标识决定是否使用缓存。设置`http header`中`Last-Modified/ETag`来实现。

前者是该资源文件最后一次修改时间。后者是服务器相应请求时，返回当前资源文件的一个唯一标识（只要资源有变化，`Etag`就会重新生成）。