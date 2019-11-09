---
title: jenkins 之构建部署
categories:
- tools
---
最近刚刚搞定 jenkins 构建部署前端项目，现在总结一波。
<!--more--> 
### 一、需求
自己业余时间搭建的项目让前后端杂糅在一起，然后共同在服务端构建部署。但是由于服务器配置有限，每次构建都会触发监控报警，所以考虑把前端构建工作放在本地 mac 机上，构建成功后再部署到服务器。

### 二、安装
#### 2.1、下载 java
- 从官网下载并安装

```
brew cask install java
```
- 查看版本信息

```
brew cask info java
```
- 检查 java 环境

```
java -version
```

#### 2.2、安装 jenkins
- 安装

```
brew install jenkins
```
- 卸载

```
brew uninstall jenkins
```
- 启用

```
jenkins
```

- 登录

```
http://localhost:8080
```
### 三、初始化
首次设置项目：unlock Jenkins（administrator password）-> Customize Jenkins（install suggested plugins）—> Create First Admin User（用户名、密码、确认密码、全名、电子邮件地址）—> Jenkins is ready
#### 3.1、安装时查看解锁密码
<img src="/assets/tools/jenkins/install.png">

由于安装时没有截图，这里使用网上的截图，也可以使用命令行查看：
```
cat /Users/zhangshaoyu/.jenkins/secrets/installAdminPassword
```
#### 3.2、解锁
<img src="/assets/tools/jenkins/unlock.png">

#### 3.3、插件安装

<img src="/assets/tools/jenkins/plugin.png">

<img src="/assets/tools/jenkins/failure.png">

这里插件安装时可能有部分插件会安装失败，可以进行下一步，到最后再重新安装回来

#### 3.4、推荐插件
除了默认推荐插件以外，还有一些比较好的插件：
- Role-based Authorization Strategy 角色管理插件
- Extended Choice Parameter 项目参数选择插件
- Git Parameter 可用于获取 git 项目分支等信息作为参数
- Email Extension Plugin 构建成功后进行邮件推送
- Publish Over SSH 用于推送静态资源包到服务器

### 四、构建
#### 4.1、基本信息

<img src="/assets/tools/jenkins/general.png">

#### 4.2、源码信息

<img src="/assets/tools/jenkins/source.png">

#### 4.3、构建脚本

<img src="/assets/tools/jenkins/build.png">

### 五、部署
#### 5.1、全局 ssh 配置

<img src="/assets/tools/jenkins/global.png">

#### 5.2、部署配置

<img src="/assets/tools/jenkins/deploy.png">

### 六、部署
#### 6.1、nginx 配置
```
server {
  listen 80;
  server_name admin-coupon.master-ss.cn;
  rewrite ^(.*) https://$host$1 permanent;
}

server {
  listen 443;
  server_name admin-coupon.master-ss.cn;
  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/1_admin-coupon.master-ss.cn_bundle.crt;
  ssl_certificate_key /etc/nginx/conf.d/ssl/2_admin-coupon.master-ss.cn.key;
  ssl_session_timeout 5m;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;#按照这个套件配置
  ssl_prefer_server_ciphers on;

  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header X-Nginx-Proxy true;
   
    root /usr/share/nginx/coupon_admin_dist;
    index index.html;
    try_files $uri /index.html;
  } 
}
```