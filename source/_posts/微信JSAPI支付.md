---
title: 微信JSAPI支付
tags: [微信支付]
date: 2026-01-17
---

## 微信JSAPI支付

需要准备的东西:
- 商户:
1. appid:商户号
2. apiv2:apiv2的支付密钥
3. jspai:需要支付的域名 xx.domain.com

- 公众号:
1. appid:公众号appid
2. appsecret:开发者密钥
3. 配置支付授权目录:xx.domain.com和上面商户配置的一致

> 注意: 商户号认证300一年,公众号认证300一年,公众号认证后可以申请商户号,但是商户号认证后不可以申请公众号,并且需要关联上才可以

可以参考[微信支付](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_3)

## 微信JSAPI支付流程

1. 商户后台生成订单,返回订单号和支付参数
2. 前端调用微信支付接口,支付参数为上一步返回的参数
3. 微信支付接口返回支付结果
4. 商户后台查询支付结果
5. 支付成功后,

> 这里有个坑:官方给的JS-SDK地址是 http://res.wx.qq.com/open/js/jweixin-1.6.0.js
这里并不方便使用所以这里可以使用 `npm install weixin-js-sdk`

```js
function onBridgeReady() {
    WeixinJSBridge.invoke('getBrandWCPayRequest', {
        "appId": "wx2421b1c4370ec43b",     //公众号ID，由商户传入     
        "timeStamp": "1395712654",     //时间戳，自1970年以来的秒数     
        "nonceStr": "e61463f8efa94090b1f366cccfbbb444",      //随机串     
        "package": "prepay_id=wx21201855730335ac86f8c43d1889123400",
        "signType": "RSA",     //微信签名方式：     
        "paySign": "sign" //微信签名 
    },
    function(res) {
        if (res.err_msg == "get_brand_wcpay_request:ok") {
            // 使用以上方式判断前端返回,微信团队郑重提示：
            //res.err_msg将在用户支付成功后返回ok，但并不保证它绝对可靠，商户需进一步调用后端查单确认支付结果。
        }
    });
}
if (typeof WeixinJSBridge == "undefined") {
    if (document.addEventListener) {
        document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
    } else if (document.attachEvent) {
        document.attachEvent('WeixinJSBridgeReady', onBridgeReady);
        document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
    }
} else {
    onBridgeReady();
}
```

> 注意事项 1.签名用的noncestr和timestamp必须与wx.config中的nonceStr和timestamp相同。2.签名用的url必须是调用JS接口页面的完整URL。3.出于安全考虑，开发者必须在服务器端实现签名的逻辑。

下面展示一下如何在服务器端实现签名
```java
PayJsSDK payJsSDK = new PayJsSDK();
SortedMap<String, Object> params = new TreeMap<>();
long timestamp = System.currentTimeMillis() / 1000;
params.put("appId", this.appId);
params.put("timeStamp", String.valueOf(timestamp));
params.put("nonceStr", payOrder.getId());
// prepay_id 请求微信下单后返回的
params.put("package", "prepay_id=" + prepay_id);
params.put("signType", "MD5");

StringBuilder url = new StringBuilder();
for (String key : params.keySet()) {
    url.append("&").append(key).append("=").append(params.get(key));
}
//log.info("对参数按照key=value的格式，并按照参数名ASCII字典序排序生成字符串：{}", url.toString());
url = new StringBuilder(url.substring(1) + "&key=" + payParam.getSign());
//log.info("连接密钥key：{}", url.toString());
// 开始生成sign
String signature = DigestUtils.md5Hex(url.toString());
//log.info("生成sign并转成大写：{}", signature);
payJsSDK.setAppId(this.appId);
payJsSDK.setTimestamp(String.valueOf(timestamp));
payJsSDK.setNonceStr(payOrder.getId());
payJsSDK.setSignature(signature.toUpperCase());
payJsSDK.setPrepayId(prepay_id);
```

以上就是微信JSAPI支付主要注意的点,其他的可以参考[微信支付](https://pay.weixin.qq.com/doc/v3/merchant/4012062524)