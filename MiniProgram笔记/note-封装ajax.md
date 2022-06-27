# 封装ajax

```js
// 封装前
// index.js
wx.request({
    url: 'http://localhost:2999/category/list/0',
    success: (res) => {
        if (res.statusCode === 200) {
            let { code, data, msg } = res.data;
            switch (code) {
                case 200:
                    this.setData({ listMain: data });
                    break;
                case 199:
                case 401:
                case 404:
                case 500:
                    console.log(msg);
            }
        } else {
            console.log(res.errMsg);
        }
    }
});

// 封装后
// app.js
App({
    onLaunch: function() {
        ...
        this.globalData = {baseUrl: 'xxx'};
    },
   http: function(userOptions) {
       userOptions.url = this.globalData.baseUrl + userOptions.url;
       userOptions.method = userOptions.method || "GET";
       userOptions.header = Object.assign({}, {Authorization: wx.getStorageSync('token')}, userOptions.header || {});
       userOptions.timeout = 3000;// 设置超时时间，超时将进入fail
       wx.showLoading({title: '加载中..'});// 发送ajax之前打开loading
       return new Promise((resolve, reject) => {
           wx.request({
               ...userOptions,
               success: res => {
                   if(res.statusCode === 200) {
                       let {code, data, msg} = res.data;
                       switch(code) {
                           case 200:
                               resolve(data);
                               // 事实上每次与服务器交互，都会返回新的token。需要更新token来延长过期时间
                               break;
                           case 199:
                           case 401:
                           case 404:
                           case 500:
                               wx.showToast({title: msg});
                               reject(msg);
                       }
                   } else {
                       wx.showToast({title: res.errMsg});
                       reject(res.errMsg);
                   }
               },
               complete: () => wx.hideLoading(),// 关闭loading
               fail: error => {
                   wx.showToast({title: '服务器连接超时'});
                   reject(error.errMsg);
               }
           });
       });
   } 
});
// index.js
const app = getApp();
Page({
    ...
    onReady: async function() {
        let listMain = await app.http({url: 'xxx'});
        this.setData({listMain});
    }
});
```

