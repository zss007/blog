---
title: frp 之内网穿透
categories:
- project
---
刚搞定内网穿透，成就感爆棚，之前一直都是用[别人的服务](http://www.ngrok.cc/)，现在终于自己搞定了。
<!--more--> 
### 一、说明
#### 1.1、优点
- 在不部署的情况下就能够演示网站
- 在移动设备上查看本地代码运行效果
- 使用固定域名地址来映射本地效果
- 方便开发对接第三方服务应用，比如微信公众号等

#### 1.2、准备
- 公网服务器 1 台，需备案
- 内网电脑

### 二、实现流程
#### 2.1、下载 frp
下载地址是 https://github.com/fatedier/frp/releases ，分别下载`frp_xxx_linux_amd64.tar.gz`和 `frp_xxx_darwin_amd64.tar.gz`。xxx 是版本号，推荐最新版本。因为本人使用 mac 电脑，如果是 windows 系统，则需下载`frp_xxx_windows_amd64.zip`。
#### 2.2、服务端启动
新建目录`mkdir -p /usr/local/frp`，上传`frp_xxx_linux_amd64.tar.gz`至服务器该目录下，解压`tar -zxvf  frp_xxx_linux_amd64.tar.gz`，进入解压目录`cd frp_xxx_linux_amd64`，移除`frpc`文件夹、frpc.ini、frpc_full.ini。
然后进行配置，`vi ./frps.ini`
```
[common]
bind_port = 7000
vhost_http_port = 3000
```
其中`bind_port`端口用来和客户端进行通信，`vhost_http_port`为服务端`http`服务端口。
保存然后启动服务`./frps -c ./frps.ini`，这是前台启动，后台启动命令为`nohup ./frps -c ./frps.ini >/dev/null 2>&1 &`
如果需要关闭这个进程，则获取进程`ps -ef|grep frps`，然后关闭`kill -9 pid`。
#### 2.3、客户端启动
首先解压`frp_xxx_darwin_amd64.tar.gz`或`frp_xxx_windows_amd64.zip`，然后删除`frps`文件夹、frps.ini、frps_full.ini。
再进行配置，`vi ./frpc.ini`
```
[common]
server_addr = xx.xx.xx.xx
server_port = 7000

[web]
type = http
local_port = 7001
custom_domains = xx.xx.cn
```
其中 server_addr 为公用服务器 ip 地址，server_port 与服务端 bind_port 一致，local_port 为内网 web 服务的端口号，custom_domains 为所绑定的公网服务器域名，一级、二级域名都可以。
保存然后启动服务`./frpc -c ./frpc.ini`。
#### 2.4、nginx 请求代理
目前还需要在域名下带端口号才能够访问，而很多对接的第三方服务不允许域名下带端口号，所以用 nginx 做下代理工作。
推荐新增二级域名，然后配置 nginx
```
upstream frps {
  server 127.0.0.1:3000;
}

server {
  listen 80;
  server_name frps.xx.xx;
  
  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header X-Nginx-Proxy true;
   
    proxy_pass http://127.0.0.1:3000; 
    proxy_redirect off;
  }
}
```
重启 nginx 后便能够使用该二级域名访问本地服务了！！！