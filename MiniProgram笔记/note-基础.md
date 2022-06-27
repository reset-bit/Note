[toc]

# 目录解析

```
- images
- pages 所有页面文件
	- index
		- .js 描述页面逻辑
		- .json 描述页面配置
		- .wxml 描述页面结构
		- .wxss 描述页面样式
- app.json 全局配置文件
	{
		"entryPagePath": '', 强行指定入口页面
		"pages":[ 页面配置节点
			'pages/index/index'
		],
		"window": {...}, 窗口相关属性，设置状态栏、导航条、标题、窗口背景色
		"tabBar": {...}, 导航栏相关
	}
- project.config.json 小程序配置文件
```

> 自定义tabBar参考：`https://developers.weixin.qq.com/miniprogram/dev/framework/ability/custom-tabbar.html`
>
> 页面数据更新失败可尝试清除缓存

# 分页面

## WXML

```html
<block>相当于vue-template react-空标签</block>

<!-- v-for -->
<!-- 自定义遍历对象与下标名称，下标从0开始 -->
<text wx:for="{{name}}" wx:for-item="myItem" wx:for-index="myIndex">{{myItem}}</text>
<!-- 0 1 2 3 4 5 6 7 8 -->
<text wx:for="{{9}}">{{item}}}</text>
<!-- item.id作为key -->
<text wx:for="{{arr}}" wx:key="id">{{item.name}}</text>
<!-- 99乘法表 -->
<view wx:for="{{9}}" wx:for-item="i">
  <text wx:for="{{i + 1}}" wx:for-item="j">{{j + 1}} * {{i + 1}} = {{(i + 1) * (j + 1)}} </text>
</view>

<!-- v-show -->
<text hidden="{{isHidden}}">demo</text>

<!-- v-if -->
<text wx:if="{{score >= 90}}">优秀</text>
<text wx:elif="{{score >= 80}}">良好</text>
<text wx:else>及格</text>

<!-- 绑定事件 -->
<button bindtap="tapHandler" data-num="100">tap</button>
<input type="text" value="{{name}}" bindinput="inputHandler" />
<script>
	inputHandler: function(e) {this.setData({name: e.detail.value});}// 类似react，合并data
</script>
```

## WXS

WXML `{{}}`中不允许出现函数调用，只允许调用WXS模块中的方法和变量

WXS是在WXML中调用的一个单独的高效率**模块脚本**。该脚本中只能使用CommonJS语法，不能使用ES6及其他高阶JS语法、不能使用页面数据及方法

```html
<wxs module="wxs">
	function strToArr(str, seperator) {return str.split(seperator);}
    // module.exports.strToArr = strToArr;
    module.exports = {
    	strToArr: strToArr
    };
</wxs>
<picker mode="region" value="{{wxs.strToArr(address.region, ' ')}}">{{address.region || '请选择'}}</picker>

<!-- 导入导出，设存在comm.wxs -->
<wxs src="../../comm.wxs" module="g"></wxs>
```

## WXSS

- 使用rpx，默认750rpx
- 不支持高阶选择器
- 尽量避免浮动
- `<image></image>`显示图片默认宽高320*240；背景图片只能显示网络图片绝对地址/本地图片base64位

## JS

页面数据结构应当参考**数据库中对应数据**的形式

### 页面跳转

小程序内部维护max=5的**页面栈**，`navigateTo-push navigateBack-pop`。进栈会导致页面渲染，非栈顶页面会隐藏，出栈会导致页面销毁。使用`getCurrentPage()`获取当前页面栈

```html
<!-- 组件跳转 -->
<navigator url="/pages/list/list">跳转非tabBar页面</navigator>
<navigator url="/pages/profile/profile" open-type="switchTab">跳转tabBar页面</navigator>
<navigator url="/pages/list/list" open-type="redirect">redirect</navigator>
<navigator url="/pages/list/list" open-type="reLaunch">reLaunch</navigator>
<navigator open-type="navigateBack" delta="2">navigateBack</navigator>
<navigator open-type="exit">exit</navigator>

<!-- API跳转 -->
<button bindtap="goToTabBarPage">跳转tabBar页面</button>
<script>
    goToNotTabBarPage: function() {
    	wx.navigateTo({ url: "/pages/list/list" });
    }
    // 下略外层函数
    wx.switchTab({ url: "/pages/profile/profile" });// 同一时间段内只能有一个在栈底的tabBar页面
    wx.redirectTo({ url: 'xxx' });
    wx.reLaunch({ url: 'xxx' });// 强行清空页面栈，并添加url对应页面
    wx.navigateBack({ delta: '2' });// 栈中没有页面仍调用本函数，将退出小程序
</script>
```

### 页面共享数据

```js
/*
	1-url
	缺：类型丢失，不能传递复杂数据
	list.wxml:
		<navigator url="/pages/detail/detail?id=1">go</navigator>
	detail.js:
		onload: function(options) {console.log(options.id);}
		
	2-storage
	有同步和异步两种模式，一般使用同步模式
	优：无类型丢失，支持多页面传递数据
	缺：手动清除
	list.js:
		try{ wx.setStorageSync('id', 100); }
		catch(e) {}
	detail.js:
		try { 
			wx.getStorageSync('id');
			wx.removeStorage('id');
			wx.clearStorage();
         }
		catch(e) {}
		
	3-app globalData
	优：无类型丢失，支持多页面共享数据
	缺：任何页面可读可写，仅适用于多页面共享
	app.js:
		App({
			onLaunch: function() {this.globalData = {num: 0}}
		});
	list.js:
		getApp().globalData.num = 99;
	detail.js:
		console.log(getApp().globalData.num);
		
	4-eventChannel
	优：适用于两个页面间产生动作并传值
	address.js:
		beginUpdate: function(e) {
			let address = this.data.list.find(item => item.id === e.currentTarget.dataset.id);
			wx.navigateTo({
				url: '/pages/address/edit/edit',
				event: {// 监听子页面事件
					add: address => {console.log(address);}
				},
				success: res => {
					res.eventChannel.emit('beginUpdate', address);// 发射事件并传值
				}
			});
		}
	edit.js:
		onload: function(options) {
			let eventChannel = this.getOpenerEventChannel();// 获取开启本页面的页面的eventChannel
			eventChannel.on('beginUpdate', address => {// 监听事件
				this.setData({address: {...address}});
			});
		}
		save: function() {
			let eventChannel = this.getOpenerEventChannel();
			eventChannel.emit(this.data.address.id ? 'update' : 'add', this.data.address);// 向主页面发射事件
			wx.navigateBack();
		}
*/
```

### 生命周期

视图层+逻辑层双线程，二者相互独立，基于消息通信将内容转变为字符串，不会相互阻塞。

视图层每个页面使用`webView`分别渲染。逻辑层不能直接操作dom和获取dom。

线程之间通信统一使用微信客户端（native）中转。

> webView是一个基于webkit的引擎，可以解析dom元素，与浏览器展示页面的原理相同。与原生app需要下载进行更新相比，webView可通过部署指定文件完成更新

```js
// App
onLaunch()// 加载完毕调用，仅执行一次
onShow()// 小程序启动或加载至前台调用
onHide()

// page
onLoad()// 
onShow()
onReady()// 页面初次渲染完成时调用，每个页面仅执行一次
onHide()
onUnload()

// component
lifetimes: {
    created()// 不能调用setData()
    atteched()// 组件挂载调用，发送ajax
    deteched()// 组件卸载调用
    moved()
    ready()
}
pageLifeTimes: {
    show()
    hide()
    resize()
}
```

# 使用第三方包

```js
// lin-ui
// 本地设置-使用npm模块
npm init -y// 当前目录创建package.json，全部默认选项
npm install lin-ui
// 工具-构建npm 将npm组价构建为微信小程序组件
```

