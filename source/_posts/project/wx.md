---
title: 微信公众号开发
categories:
- project
---
总结一波微信公众号开发的流程。
<!--more--> 
### 一、基础知识
#### 1.1、企业号
为企业或组织提供移动应用入口，帮助企业建立与员工、上下游供应链及企业应用间的连接（管理全学校所有学院，团委，学生处各个部门上班人员的考勤，活动进程等，目前钉钉是更好的选择）
#### 1.2、订阅号
适合于个人、小团队，主要是用于信息传播，帮助管理用户以及用户互动。比如撰写文章，咨询传播，消息定制等等（比如管理一个班级，一个学院的信息订阅，通知和互动）
#### 1.3、服务号
企业和组织，提供更强大的业务服务与用户管理能力。比如支付，智能接口（管理全学校的水果商店或者打印店，可以直接支付送货上门，及时推送一些特价水果）
#### 1.4、认证
一般是需要你有一个开户过的企业，可以以法人身份折腾下开一个小公司，建议是认证一下
#### 1.5、订阅号和服务号区别
- 出现位置不同；
- 单月发送消息数量不同，订阅号可以一天一篇，服务号一个月最多4篇；
- 订阅号没有一些接口和功能

### 二、公众号开发
#### 2.1、配置微信公众号后台
- URL 用来接收微信消息和事件；
- Token 任意填写，用作生成签名；
- EncodingAESKey 手动填写或随机生成，用作消息体加解密密钥

#### 2.2、公众号验证
在公众号平台上设置服务器配置后，微信会用 GET 请求配置中的服务器地址，含以下参数：
- signature 微信加密签名
- timestamp 时间戳
- nonce 随机数
- echostr 随机字符串

验证流程如下：
- 将token、timestamp、nonce三个参数进行字典序排序
- 将三个参数字符串拼接成一个字符串进行sha1加密
- 开发者获得加密后的字符串可与signature对比，标识该请求来源于微信，若确认此次GET请求来自微信服务器，则返回echostr参数内容
- 服务器配置启用大概需要10分钟的缓存时间，停用大概5分钟的反应时间

#### 2.3、被动回复消息
- 处理POST类型的控制逻辑，接收这个XML的数据包
- 解析这个数据包（获得数据包的消息类型或者事件类型）
- 拼装好我们定义好的消息
- 包装成XML的格式
- 在5秒内返回回去

#### 2.4、access_token
- access_token是公众号的全局唯一接口调用凭据，公众号调用各接口时都需使用access_token。开发者需要进行妥善保存。access_token的存储至少要保留512个字符空间
- access_token的有效期目前为2个小时，需定时刷新，重复获取将导致上次获取的access_token失效
- 服务器需要根据有效时间提前去刷新新access_token。在刷新过程后的5分钟内，新老access_token都可用
- 建议统一获取和刷新access_token，其他服务器所使用的access_token均来自于该中控服务器，不应该各自去刷新，否则容易造成冲突，导致access_token覆盖而影响业务

#### 2.5、jssdk
- jsapi_ticket是公众号用于调用微信JS接口的临时票据，有效期为2小时
- jsapi_ticket通过access_token来获取，获取的api调用次数非常有限，须在服务全局缓存
- 绑定域名
- 引入JS文件
- 通过config接口注入权限验证配置
- 通过ready接口处理成功验证
- 通过error接口处理失败验证

### 三、vue 单页调用 JSSDK
#### 3.1、定义 vuex 缓存
```
export default new Vuex.Store({
  state: {
    wxSignUrl: '',
  },
  mutations: {
    // IOS仅记录第一次进入页面时的URL，IOS微信切换路由实际URL不变，只能使用第一进入页面的URL进行签名
    setWechatSignUrl(state, value) {
      if (tools.isIos() && state.wxSignUrl !== '') {
        return
      }
      state.wxSignUrl = value
    }
  },
  getters: {
    getWechatSignUrl: state => state.wxSignUrl
  }
})
```
#### 3.2、路由守卫内触发更新签名
在IOS上，无论路由切换到哪个页面，实际真正有效的的签名URL是【第一次进入应用时的URL】。这里保证IOS是使用当前页面URL，Android是使用目标路由完整地址再加上域名
```
// 获取真实有效微信签名URL
function getWechatSignUrl(to) {
  if (tools.isIos()) {
    return window.location.href
  } else {
    return process.env.VUE_APP_DOMAIN + to.fullPath
  }
}

router.beforeEach((to, from, next) => {
  store.commit('setWechatSignUrl', getWechatSignUrl(to))
  next()
})
```
#### 3.3、定义全局 mixin
```
export default {
  data() {
    return {
      firstIn: true,
    }
  },
  mounted() {
    this.initWechatShareConfig()
  },
  activated() {
    if (!this.firstIn) {
      this.initWechatShareConfig()
    }
    this.firstIn = false
  },
  methods: {
    initWechatShareConfig() {
      const u = navigator.userAgent.toLowerCase()
      if (u.match(/MicroMessenger/i) != 'micromessenger') return
      // 获取服务端签名配置信息
      this.$service({
        host: 'coupon',
        api: 'getWxConfig',
        option: {
          data: {
            url: this.$store.getters['getWechatSignUrl']
          },
        },
      }).then(res => {
        if (res.code === 200) {
          this.$tools.initWx(res.data, this.$route.meta.title, this.$route.fullPath)
        } else {
          this.$toast({
            position: 'top',
            message: res.message || '请求失败',
          })
        }
      }).catch(() => {
        this.$toast.clear()
      })
    },
  }
}
```
#### 3.4、定义初始化工具方法
```
initWx: function (wxConfig, title, path) {
  window.wx.config({
    debug: process.env.NODE_ENV === 'production' ? false : true, // 调试模式,
    appId: process.env.VUE_APP_APPID, // 必填，服务号的唯一标识
    timestamp: parseInt(wxConfig.timestamp), // 必填，生成签名的时间戳
    nonceStr: wxConfig.noncestr, // 必填，生成签名的随机串
    signature: wxConfig.signature,// 必填，签名，见附录1
    jsApiList: ['updateAppMessageShareData', 'updateTimelineShareData'] // 必填，需要使用的JS接口列表
  })

  window.wx.ready(function () {
    // 调用jssdk方法
    ...
  })

  window.wx.error(function (err) {
    console.log('wx error: ', err)
  })
}
```
#### 3.5、页面继承 mixin
```
export default {
  mixins: [mixins],
}
```
