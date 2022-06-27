

# 规范
1. 通栏：与窗口同宽；最小宽度为版心宽度 `class=tong`；版心：宽度固定；水平居中 `class=w`
2. 缩写：
```
	nav==>navigator
	tr=>table row
	reg=>regular
```
3. color：每两位是`red\green\blue`的16位简写
```
	#fff 白色
	#eee 网站底色
	#333 字体颜色
```
4. 重要的CSS样式写在前面

5. **z-index**仅对定位元素有效

6. script标签写在head标签中，或body标签中最后
	
	> 浏览器由上到下解析dom元素，放在body标签最后，防止找不到该元素

7. public/static/assets下分css、js，css下reset.css、public.css

8. 快速引用其他html文件：使用`<iframe></iframe>`；其中被引用页面a标签跳转`<a href="javascript:window.parent.location.href='xxx'"></a>`

9. 单向跳转`window.location.replace();`，如登录、付款页面；获取前身页面`document.referrer`

10. 浏览器规定，移动端页面100vh\*100vw；dp虚拟像素，在不同的像素密度的设备上会自动匹配

12. **HTML 超文本标记语言**，被设计用来显示数据，核心在于数据外观；**XML可扩展标记语言**，被设计用于结构化、传输和存储数据，核心在于数据内容

13. 验证码实现方式：客户端保存图片形式的验证码，服务器端保存cookie对应的图形验证码的正确文本。客户端表单提交时提交到服务器端验证。（将正确的验证码放在客户端是违背验证码初衷的，爬虫或恶意程序可以模拟表单提交来达到攻击目的）

13. 内部私有方法，不被dom调用的方法以_开头

14. js中以`camel case`驼峰命名法最为自然，数据库以`under score case`下划线命名法最为自然。后端可通过mybatis注解进行命名转换注入。

    ```js
    // 驼峰转下划线命名法
    let getUnderScoreCaseFormCamelCase = function(str) {
        let arr = str.split('');
        let res = arr.map(item => {
            if(/[a-zA-Z]/.test(item) && item.toUpperCase() === item) {
                return '_' + item.toLowerCase();
            } else {
                return item;
            }
        });
        return res.join('');
    };
    console.log(getUnderScoreCaseFromCamelCase('helloWorld!'));
    ```

15. 

