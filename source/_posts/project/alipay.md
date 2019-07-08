---
title: alipay 之电脑网站支付
categories:
- project
---
近期有接入支付宝的电脑网站支付 api，遇到一些需要注意事项，记录下来。
<!--more--> 
### 一、准备
#### 1.1、申请接入
可以查看官方文档 https://b.alipay.com/signing/productDetailV2.htm?productId=I1011000290000001000
#### 1.2、接入文档
可以查看官方文档 https://docs.open.alipay.com/270/105898
### 二、开发流程
#### 2.1、sdk 获取
这里以 nodejs 的 sdk 为例，官方在 npm 已发布 [alipay-sdk](https://www.npmjs.com/package/alipay-sdk) 包
#### 2.2、接口调用
官方文档可查看 https://www.yuque.com/chenqiu/alipay-node-sdk/page_api ，特别注意电脑网站支付属于页面类的接口调用，不要弄混了。以 egg 为例：
```
<!-- app.js -->
const AlipaySdk = require('alipay-sdk').default
const fs = require('fs')

module.exports = app => {
  app.beforeStart(async () => {
    // 将 AlipaySdk 实例挂载到 app 上
    app.alipaySdk = new AlipaySdk({
      appId: 'xxxx',
      privateKey: fs.readFileSync('./rsa_private_key.pem', 'ascii'),
    })
  })
}

<!-- app/controller/xxx.js -->
const AlipayFormData = require('alipay-sdk/lib/form').default
async alipay() {
  const { ctx, } = this

  const { amount, tradeNo, invoice, notifyUrl, } = ctx.request.body

  const formData = new AlipayFormData()
  // 调用 setMethod 并传入 get，会返回可以跳转到支付页面的 url
  formData.setMethod('get')
  formData.addField('notifyUrl', notifyUrl)
  formData.addField('bizContent', JSON.stringify({
    outTradeNo: tradeNo,
    passbackParams: invoice,
    totalAmount: amount,
    qrPayMode: '2',
    productCode: 'FAST_INSTANT_TRADE_PAY',
    subject: 'xxx',
  }))
  const result = await this.app.alipaySdk.exec('alipay.trade.page.pay', {}, {
    formData: formData
  })

  ctx.body = {
    code: 200,
    status: 200,
    url: result,
  }
}
```
#### 2.3、页面跳转
这里有个地方需要特别注意，本人开发中在 vscode 的控制台上直接打印了跳转 url，然后点击在浏览器上查看会报错 `错误代码 invalid-signature 错误原因: 验签出错，建议检查签名字符串或签名私钥与应用公钥是否匹配，网关生成的验签字符串为`，查阅了大量资料才在 [issue](https://github.com/alipay/alipay-sdk-nodejs/issues/25) 上看到答案。正确的做法是拷贝控制台上的链接，然后手动复制到浏览器地址栏，直接点击会被 decodeURIComponent 处理导致验签失败。