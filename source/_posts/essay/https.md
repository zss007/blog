---
title: 网络之 https 协议
categories:
- essay 
---
https 是建立在 SSL(Secure Sockets Layer 安全套接层)上的网络安全协议，最初由 NetScape 公司提出，之后由 IETF 标准化为 TLS(Transport Layer Security 安全传输层协议），其端口为 443。
<!--more-->
### 一、http 困境
http 协议是明文传递内容，一旦在网络被人监控，数据可能一览无余地展现在中间的窃听者面前。为此我们需要将数据加密后再进行网络传输，这样即使被截取和窃听，窃听者也无法知道数据的真实内容是什么。
### 二、加密
让我们设想这样一个情景：海绵宝宝想和蟹老板秘密协商新的蟹黄堡配方，如果让你来设计加密过程，你有几种方法呢？
#### 2.1、对称加密
海绵宝宝和蟹老板约定一个密钥，传输和解读都通过这个密钥解密，这个密钥称为公钥。
<img src="/assets/essay/https-symmetry.png" />
#### 2.2、非对称加密
小心谨慎的蟹老板有一个只有自己知道的密钥(称为私钥)和公钥，他把公钥发给海绵宝宝，海绵宝宝可以通过公钥解密私钥加密的消息，但是公钥加密的消息只有私钥能解开，这在一定程度上做到了单向安全。
<img src="/assets/essay/https-asymmetry.png" />
#### 2.3、协商
那么HTTPS中用的哪种加密方式呢？
如果采用对称加密，那么蟹老板和所有人都拥有同一个密钥，这无异于没有加密！！！
<img src="/assets/essay/https-negotiate1.png" />
所以蟹老板需要和每个人协商一个密钥，每个人互不相同，这样就能保证加密了。
<img src="/assets/essay/https-negotiate2.png" />
在HTTPS中，这个协商的过程一般是用非对称加密来进行的。客户端一旦得到了真的服务器公钥，往服务端传消息就是安全的。因为只有服务端的私钥才能解密公钥加密的数据。
但是，客户端可没那么容易得到真正的公钥，因为发送公钥的过程存在被别人调包的可能性，这就是传说中的中间人攻击。
#### 2.4、中间人攻击
让我们回到情景中。
痞老板听闻消息，企图在协商过程通过中间人的方式截取蟹黄包配方。
<img src="/assets/essay/https-middleman.png" />
如图，蟹老板想告诉海绵宝宝自己的公钥，此时痞老板出现，替换了蟹老板的公钥，海绵宝宝收到一个来自痞老板的公钥并以为是蟹老板的，由此在之后的传输中痞老板便可以轻松的浏览蟹老板和海绵宝宝的所有沟通内容了。为了防止痞老板窃听，那这个协商的过程也必须加密，这样下去协商加密也要加密，……问题没有穷尽。那怎么办呢？
#### 2.5、HTTPS 数字证书
现在美人鱼战士和企鹅男孩登场了，他们保证作为一个权威中间机构为大家提供认证服务，负责为大家发放统一的公钥。
<img src="/assets/essay/https-ca.png" />
蟹老板先将自己要传输的公钥给美人鱼战士，美人鱼战士用自己的私钥加密后返回给蟹老板。这样大家只要能通过这个公钥解密蟹老板发来的密文，提取蟹老板的公钥，就能保证这个公钥不是来自痞老板。这个时候即便痞老板想替换公钥，伪造的公钥也不能用美人鱼战士的公钥解开了。
这就是数字证书。服务器通过CA认证得到证书，这个证书包含CA的私钥加密后的服务器公钥，客户端用预先存储在本地的CA公钥即可解密得到服务器的公钥，从而避免公钥被替换。
#### 2.6、HTTPS 数字签名
如果你是痞老板，你有什么方法可以再次窃取消息呢？
显然不能通过简单替换公钥来窃取了，海绵宝宝只能解开美人鱼战士颁发的证书，那我也去申请一个证书，直接替换整个证书不就可以从而替换掉公钥了吗？
<img src="/assets/essay/https-sign.png" />
企鹅男孩发现了这个问题，他决定在证书里面添加一些额外的信息以供验证。现在企鹅男孩颁发的证书格式如下：
```
来自：蟹堡王
加密算法：MD5
公钥：xxxx（已通过私钥加密，可通过公钥解密）
```
海绵宝宝只要通过企鹅男孩的公钥提取到“蟹堡王”+“MD5” 计算出一个公钥，与证书内的公钥进行对比，就可以验证证书是否经过替换，如下图。
<img src="/assets/essay/https-md5.png" />
带有签名的证书
<img src="/assets/essay/https-sign-md5.png" />
对比发现证书被替换
虽然痞老板依旧可以截取证书，但是他却不能替换其中任何的信息，如下：
```
来自：痞老板工厂
加密算法：MD5
公钥：djawdn888
```
此时海绵宝宝可以发现来自不是蟹堡王而拒绝信任，并且
```
痞老板工厂 + MD5 !== djawdn888
```
现在假设`蟹堡王+ MD4 = aaaa`，那只要修改公钥为aaaa不就可以通过海绵宝宝的验证了吗？痞老板的证书修改如下：
```
来自：蟹堡王
加密算法：MD4
公钥：aaaa
```
实际上并不能，尽管可以修改公钥，但是前面提到，公钥经过企鹅男孩的私钥加密，现在海绵宝宝发现用企鹅男孩的公钥打不开了！于是发现证书已经被篡改了，从而结束通讯。 这便是数字签名的意义。
#### 2.7、小结
综上，用一句话总结https：在https协议下，服务器与客户端通过非对称加密的方式协商出一个对称加密的密钥完成加密过程。其中数字证书的作用是避免公钥被替换，而数字签名的作用是校验公钥的合法性。
ps：权威机构的公钥是由操作系统和浏览器共同维护，预先存储在本地的。并由上可知 https 具有使用密文，安全性高的优点，同样的，存在协商过程低效，影响用户访问速度的缺点。
### 三、握手过程
明白了HTTPS的原理，握手过程就十分简单，总结如下：
- 客户端：发送 random1 + 支持的加密算法 + SSL Version 等信息
- 服务端：发送 random2 + 选择的加密算法 A + 证书
- 客户端：验证证书 + 公钥加密的 random3
- 服务端：解密 random3，此时两端共有 random1，random2，random3，使用这3个随机数通过加密算法计算对称密钥即可。

以上只有 random3 是加密的，所以用 random1 + 2 + 3 这3个随机数加密生成密钥。
### 四、https 服务器
#### 4.1、搭建
这里以腾讯云为例，首先申请 SSL 证书：企业型、企业型专业版、域名型、域名型免费版、增强型、增强型专业版。如果对安全性要求不是那么高的话，则使用`域名型免费版`。审批通过后下载证书压缩包（对于中小型企业，如果服务器厂商不提供免费版，也可以在自己服务器上自建 CA 机构）。
获取到证书后，将`Nginx`文件夹目录下的证书文件`1_www.domain.com_bundle.crt`、私钥文件`2_www.domain.com.key`保存到同一个目录，如 `/etc/nginx/conf.d/ssl` 目录下，修改 nginx 配置如下：
```
server {
    listen 443;
    server_name www.domain.com; #填写绑定证书的域名
    ssl on;
    ssl_certificate /etc/nginx/conf.d/ssl/1_www.domain.com_bundle.crt;
    ssl_certificate_key /etc/nginx/conf.d/2_www.domain.com.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #按照这个协议配置
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;#按照这个套件配置
    ssl_prefer_server_ciphers on;
    location / {
        ...
    }
}
```
配置完成后，请先执行命令 nginx –t 测试 Nginx 配置是否有误。若无报错，重启 Nginx 之后，即可使用`https://www.domain.com`来访问。
#### 4.2、自动跳转
对于用户不知道网站可以进行 HTTPS 访问的情况下，让服务器自动把 HTTP 的请求重定向到 HTTPS。nginx 配置如下：
```
server {
  listen 80;
  server_name www.domain.com;
  rewrite ^(.*) https://$host$1 permanent;
}
```
### 五、参考
- [海绵宝宝也懂的HTTPS](https://juejin.im/post/5c341549e51d45524860cf99?utm_source=gold_browser_extension)