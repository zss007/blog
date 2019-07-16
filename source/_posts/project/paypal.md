---
title: paypal 之网站集成
categories:
- project
---
项目要求，Web 网站需要实现 paypal 收款，废话不说，直接切入正题。
<!--more--> 
### 一、准备
#### 1.1、注册账号
在 paypal [官网](https://www.paypal.com)注册一个账号，注意要账户类型要选择企业账户，其他信息如实填写好。注意 paypal 是支持接入个体工商户的，亲测有效。
#### 1.2、调解中心
注册成功并完善资料后，账号登录要前往`调解中心`点击`前往账户限制`按钮，（完善账号信息，否则正式对接接口的时候，表单提交直接报错！！这是一个坑！！），需要提交以下文件：中国大陆居民身份证（正反面）和营业执照照片；这个审核过程一般 1 天左右的时间就可以完成。当然还需要邮箱验证等，这些都比较简单立刻能够操作完成。
#### 1.3、沙箱测试
当账户注册成功以后，paypal 会分配给开发者账号两个沙箱测试账号（一个买家账号和一个商家账号）。去 paypal [开发者账号管理端](https://developer.paypal.com/developer/accounts)查看，用上面刚刚注册的账号密码即可。
系统自动分配两个 Country 值为 C2（代表中国区）的账号，但是千万不要同时拿着这两个账号来进行沙箱收付款测试，因为国内政策原因， paypal 规定中国地区和中国地区的账户之间无法实现付款，所以新建一个沙箱测试账号，买家的余额随便填，但是买家和商家的地区一定要选择不一样。
### 二、代码开发
#### 2.1、form 表单
其实代码很简单，就一段 Html Form 表单。
```
<!-- https://www.paypal.com/cgi-bin/webscr 生产地址 -->
<!-- https://www.sandbox.paypal.com/cgi-bin/webscr  沙箱测试地址 -->
<form action="https://www.paypal.com/cgi-bin/webscr" method="post" target="_blank">
  <input type="hidden" name="cmd" value="_xclick">  <!-- name="cmd"这个参数比较重要，_xclick表示立即支付，还有_s_xclick加密按钮等 -->
  <input type="hidden" name="business" value="商家账号">
  <input type="hidden" name="item_name" value="产品名称">
  <input type="hidden" name="item_number" value="产品编号">
  <input type="hidden" name="amount" value="金额">
  <input type="hidden" name="currency_code" value="USD">  <!-- 币种 -->
  <input type='hidden' name='return' value="支付成功后网页跳转地址">
  <input type='hidden' name='notify_url' value="异步通知交易信息地址">
  <input type='hidden' name='invoice' value="发票编码">  <!-- 注意：不能提交两次一样的发票编码 -->
  <input type='hidden' name='charset' value='utf-8'>  <!-- 字符集 -->
  <input type="hidden" name="no_shipping" value="1">  <!-- 不要求客户提供收货地址 -->
  <input type="hidden" name="no_note" value="1">  <!-- 付款说明 -->
  <input type='hidden' name='rm' value='1'>  <!-- 付款完成后的返回URL的行为 -->
</form>
```
下面列举些付款变量列表：
- cmd: _xclick 表示立即购买；_xclick_subscription 表示订阅；_cart 表示购物车；s_x-click 表示加密按钮
- return: 指完成付款后客户的浏览器返回到的 URL。如果省略，则您的买家将被带到 paypal 网站
- notify_url: 用于接受 paypal 发送的关于即时付款通知的交易信息的 URL，必须是有效的 URL
- no_shipping: 买家的送货地址，省略或设为 0 则提示客户输入收货地址，1 则不要求客户提供收货地址，2 则客户必须提供收货地址
- no_note: 为付款加入说明，如果省略或设为 0，则会提示您的客户输入说明，如果设为 1，则不会提示您的客户输入说明
- rm: 付款完成后的返回 URL 的行为，只有在 return 变量被设置后才能生效。如果省略或为 0，则 GET 方法用于没有启用即时付款通知的所有购物车交易，而 POST 方法用于所有其他交易；如果为 1 并设置了return，则客户的浏览器由 GET 方法返回至返回 URL，并且不提交任何交易变量；如果为 2 并设置了 return，则客户的浏览器由 POST 方法返回至返回 URL，同时将所有可用交易变量发送至该 URL

更多详情查看[paypal付款按钮变量列表](http://www.fyhqy.com/post-150.html)或官方文档。
#### 2.2、提交支付
表单代码一提交，就链接到了 paypal 的支付界面，然后输入买家账号、密码就能完成支付；等支付成功以后，paypay 会把支付信息回调给notify_url 提供的接口地址（paypal 如果回调失败，最多回调三次）。
等沙箱测试通过以后 ，就直接切换成生产地址就好；这里不得不赞美一下paypal的沙箱搞的真不错，几乎和生产可以同步。