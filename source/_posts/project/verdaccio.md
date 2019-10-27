---
title: 搭建npm私有库
categories:
- project
---
最近使用 verdaccio 搭建了一个 npm 私有库，verdaccio 特点如下：
- 私有包托管在内部服务器中
- 项目中即可使用公用仓库上的公共包，也可使用内部服务器上的私有包
- 下载的时候公共包走公共仓库，私有包走内部服务器的私有仓库
- 只缓存下载过的包，而不是全部同步

具体搭建过程如下。
<!--more--> 
### 一、安装
verdaccio 官网地址 https://verdaccio.org/zh-CN
安装依赖包：
```
npm install -g verdaccio --unsafe-perm=true --allow-root
```
安装依赖包添加权限是因为直接安装会导致某些权限上的问题，哪怕是用 root 权限去执行
### 二、修改配置
#### 2.1、修改配置文件
配置文件路径：~/.config/verdaccio/config.yaml（PS：使用系统为 CentOS 7.5）
将 `auth: htpasswd:` 下的 max_users 改为 -1，让用户不能通过 npm adduser 注册账户
#### 2.2、手动添加账号
生成 md5 的 htpasswd 密码，有[密码生成的网站](http://www.htaccesstools.com/htpasswd-generator/)
会生成以下格式的用户名和密码
```
username:$apr1$809o3o2I$.pN83j6srvreYZA4NL8GF0
```
在`~/.config/verdaccio`下创建或编辑文件 htpasswd
写入上面字符串，每个用户占一行，现在就可以通过上面的用户名和密码访问了
### 三、启动服务
使用 pm2 管理 verdaccio，如果没有安装 pm2，需安装一下：
```
<!-- 安装 pm2 全局包 -->
npm install pm2 -g

<!-- 使用 pm2 启动 verdaccio 服务 -->
pm2 start/stop verdaccio

<!-- 查看进程启动情况 -->
pm2 ls
```
### 四、配置 nginx 反向代理
由于 verdaccio 默认是启动在 4873 端口，方便起见，配置 nginx 反向代理到该端口
如果需要查看 verdaccio 的端口号，可以用 `pm2 ls` 查看到 verdaccio 的 pid，然后使用：
```
netstat -nap | grep <pid>
```
查看占用的端口号。
然后我们去配置域名解析，然后配置 nginx，配置如下：
```
server {
  listen 80;
  server_name npm.master-ss.cn;
  location / {
    proxy_pass              http://127.0.0.1:4873/;
    proxy_set_header        Host $host;
  }
}
```
然后使用 `nginx -s reload` 重启 nginx 服务器，等域名解析生效后就能直接访问了。
### 五、使用
#### 5.1、nrm 切换镜像源
```
<!-- 全局安装 nrm -->
npm install -g nrm

<!-- 增加源 -->
nrm add masterss http://npm.master-ss.cn/

<!-- 查看可选的源 -->
nrm ls

<!-- 切换源 -->
nrm use masterss
```
#### 5.2、发布包
```
<!-- 登录 -->
npm login

<!-- 发布 -->
npm publish

<!-- 卸载发布 -->
npm unpublish
```
#### 5.3、安装包
```
<!-- 方法一：使用 --registry 添加 -->
npm install --save xxx --registry=http://npm.master-ss.cn/

<!-- 方法二：使用 nrm 切换源 -->
nrm use masterss
npm install --save xxx 
```
推荐在项目初始化的时候使用方法一，因为使用方法二切换到私有源安装的时候会比较慢。