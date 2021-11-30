cryptojs
--------

* with little modification, converted from googlecode project [crypto-js](http://code.google.com/p/crypto-js/), and keep the source code structure of the origin project on googlecode
* source code worked in both browser engines and node scripts. see also: [https://github.com/gwjjeff/crypto-js-npm-conv](https://github.com/gwjjeff/crypto-js-npm-conv)
* inspiration comes from [ezcrypto](https://github.com/ElmerZhang/ezcrypto), but my tests cannot pass with his version ( ECB/pkcs7 mode ), so I made it myself

### install

```
npm install cryptojs
```

### usage (example with [coffee-script](http://coffeescript.org/))

```coffee
Crypto = (require 'cryptojs').Crypto
key = '12345678'
us = 'Hello, 世界!'

mode = new Crypto.mode.ECB Crypto.pad.pkcs7

ub = Crypto.charenc.UTF8.stringToBytes us
eb = Crypto.DES.encrypt ub, key, {asBytes: true, mode: mode}
ehs= Crypto.util.bytesToHex eb

eb2= Crypto.util.hexToBytes ehs
ub2= Crypto.DES.decrypt eb2, key, {asBytes: true, mode: mode}
us2= Crypto.charenc.UTF8.bytesToString ub2
# should be same as the var 'us'
console .log us2
```








注意：目前该接口针对非个人开发者，且完成了认证的小程序开放（不包含海外主体）。需谨慎使用，若用户举报较多或被发现在不必要场景下使用，微信有权永久回收该小程序的该接口权限。

因为解密手机号需要：APPID、session_key、encryptedData、iv；
● session_key由接口wx.login请求到的code，再通过https://api.weixin.qq.com/sns/jscode2session?得到
● APPID
● encryptedData 由云开发接口 或 登录按钮等得
● iv  由云开发接口 或 登录按钮等得


1. 获取session_key：
调用wx.login接口，并申请https://api.weixin.qq.com/sns/jscode2session。

   onShow: function () {
        let that = this;
        // 判断是否缓存手机号
        that.data.userInfo = wx.getStorageSync("userInfo");
        if (that.data.userInfo == null) {
            wx.login({
                success(res) {
                    let code = res.code
                    //发起网络请求
                    wx.request({
                        url: 'https://api.weixin.qq.com/sns/jscode2session?&',
                        data: {
                            appid: '。。。',
                            js_code: code,
                            secret: '。。。',
                            grant_type: 'authorization_code'
                        },
                        method: 'get',
                        success(res) {
                            console.log(res)
                            that.setData({
                                session_key: res.data.session_key
                            })
                            wx.setStorageSync('session_key', res.data.session_key)
                            wx.switchTab({
                                url: '../../pages/scan/index'
                            })

                        }
                    })
                }
            })

        } else {
            wx.showLoading({
                title: '登录中' // 数据请求前loading
            })
            wx.switchTab({
                url: '../../pages/scan/index'
            })
        }

    },



2. button组件
因为需要用户主动触发才能发起获取手机号接口，所以该功能不由 API 来调用，需用 button 组件的点击来触发。

<button open-type="getPhoneNumber" bindgetphonenumber="getPhoneNumber">立即绑定</button>

// 获取手机号
    getPhoneNumber: function (e) {
        console.log(e)
        var that = this
        if (e.detail.errMsg == "getPhoneNumber:fail user deny") { //用户决绝授权  
            //拒绝授权后弹出一些提示  
        } else { //允许授权  
            let pc = new WXBizDataCrypt('wxe6105adbedd583e5', that.data.session_key);
            let data = pc.decryptData(e.detail.encryptedData, e.detail.iv);
            console.log(data.phoneNumber);
          
          
        }
    },
3. 解密：
3.1 新建WXBizDataCrypt.js
/**
 * Created by rd on 2017/5/4.
 */
// 引入CryptoJS
var Crypto = require('cryptojs-master-2/cryptojs.js').Crypto;
var app = getApp();

function RdWXBizDataCrypt(appId, sessionKey) {
  this.appId = appId
  this.sessionKey = sessionKey
}

RdWXBizDataCrypt.prototype.decryptData = function (encryptedData, iv) {
  // base64 decode ：使用 CryptoJS 中 Crypto.util.base64ToBytes()进行 base64解码
  var encryptedData = Crypto.util.base64ToBytes(encryptedData)
  var key = Crypto.util.base64ToBytes(this.sessionKey);
  var iv = Crypto.util.base64ToBytes(iv);

  // 对称解密使用的算法为 AES-128-CBC，数据采用PKCS#7填充
  var mode = new Crypto.mode.CBC(Crypto.pad.pkcs7);

  try {
    console.log(key, iv)
    // 解密
    var bytes = Crypto.AES.decrypt(encryptedData, key, {
      asBpytes: true,
      iv: iv,
      mode: mode
    });

    var decryptResult = JSON.parse(bytes);

  } catch (err) {
    console.log(err)
  }

  console.log(decryptResult)

  if (decryptResult.watermark.appid !== this.appId) {
    console.log(err)
  }

  return decryptResult
}

module.exports = RdWXBizDataCrypt


3.2 找一个解密crypto,可直接下载我提供的

