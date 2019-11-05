---
title: nginx 简介
categories:
- note 
---
Nginx 是一个高性能的 HTTP 和反向代理 web 服务器，这次来系统总结下 nginx。
<!--more-->
### 一、优点
- 支持海量高并发
- 内存消耗少
- 免费使用可以商业化
- 配置文件简单

### 二、使用
#### 2.1、版本说明
- Mainline version：开发版，主要是给广大 Nginx 爱好者，测试、研究和学习的，但是不建议使用于生产环境
- Stable version：稳定版,也就是我们说的长期更新版本。这种版本一般比较成熟，经过长时间的更新测试，所以这种版本也是主流版本
- Legacy version：历史版本，如果你需要以前的版本，Nginx 也是有提供的

#### 2.2、安装
- mac

```
<!-- 使用 homebrew 快捷安装 -->
brew install nginx

<!-- 查看软件包信息，可浏览到配置文件信息 -->
brew info nginx

<!-- 启动 nginx，注意需要加 sudo，不然会出现命令行提示启动成功实际上进程没有执行的情况 -->
sudo brew services start nginx
```
- centos

```
<!-- 安装 nginx 安装包 -->
yum install nginx

<!-- 查看软件包信息 -->
yum info nginx

<!-- 启动 nginx -->
systemctl start nginx.service
```
### 三、常用命令
- nginx -v
检测版本

- rpm -ql nginx
查看安装目录，rpm 是 linux 的 rpm 包管理工具，-q 代表询问模式，-l 代表返回列表，这样我们就可以找到 nginx 的所有安装位置了

- nginx -t
测试配置是否有语法错误

- nginx
启动 nginx

- nginx -s reload
重新载入配置文件，不用重启

- nginx -s stop
立即停止服务，无论进程是否在工作，都直接停止进程

- nginx -s quit
从容停止服务，需要进程完成当前工作后再停止

- ps aux | grep nginx
查看服务运行状态

- systemctl start nginx.service
启动 nginx

- systemctl stop nginx.service
systemctl 停止

- systemctl restart nginx.service
重启服务

- killall nginx
直接杀死进程，用于上面方法没有效果时

- netstat -tlnp
查看端口号的占用情况。默认情况下，nginx启动后会监听 80 端口，从而提供 HTTP 访问，如果 80 端口已经被占用则会启动失败

### 四、配置解读
#### 4.1、初始配置（/etc/nginx/nginx.conf）
```
#运行用户，默认即是nginx，可以不进行设置
user  nginx;
#Nginx进程，一般设置为和CPU核数一样
worker_processes  1;   
#错误日志存放目录
error_log  /var/log/nginx/error.log warn;
#进程pid存放位置
pid        /var/run/nginx.pid;
events {
    worker_connections  1024; # 单个后台进程的最大并发数
}
http {
    include       /etc/nginx/mime.types;   #文件扩展名与类型映射表
    default_type  application/octet-stream;  #默认文件类型
    #设置日志模式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;   #nginx访问日志存放位置
    sendfile        on;   #开启高效传输模式
    #tcp_nopush     on;    #减少网络报文段的数量
    keepalive_timeout  65;  #保持连接的时间，也叫超时时间
    #gzip  on;  #开启gzip压缩
    include /etc/nginx/conf.d/*.conf; #包含的子配置项位置和文件
}
```
#### 4.2、默认配置（/etc/nginx/conf.d/default.conf）
```
server {
    listen       80;   #配置监听端口
    server_name  localhost;  //配置域名
    #charset koi8-r;     
    #access_log  /var/log/nginx/host.access.log  main;
    location / {
        root   /usr/share/nginx/html;     #服务默认启动目录
        index  index.html index.htm;    #默认访问文件
    }
    #error_page  404              /404.html;   # 配置404页面，还可以使用外部地址，如 http://abc.com
    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;   #错误状态码的显示页面，配置后需要重启
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}
    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}
    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```
### 五、常用功能
#### 5.1、访问控制
```
# 简单访问控制
location / {
    # allow 是允许访问，在同一个块下的两个权限指令，先出现的设置会覆盖后出现的设置
    allow  45.76.202.231;
    # deny 是禁止访问，如果 123.9.51.42/200 表示 42-200 都被禁止，all 表示禁止所有 IP
    deny   123.9.51.42;
}

# 复杂访问控制权限匹配
location =/img{
    # = 号代表精确匹配，使用了 = 后是根据其后的模式进行精确匹配
    allow all;
}
location =/admin{
    deny all;
}

// 使用正则表达式设置访问权限，以 ~ 开头
location ~\.php$ {
    deny all;
}
```
#### 5.2、配置虚拟主机
```
# 基于端口号：原理就是Nginx监听多个端口，根据不同的端口号，来区分不同的网站
# 可以直接配置在主文件里 etc/nginx/nginx.conf 文件里
# 也可以配置在子配置文件里 etc/nginx/conf.d/default.conf
# 还可以再新建一个文件，只要在 conf.d 文件夹下就可以了
server{
    listen 8001;
    server_name localhost;
    root /usr/share/nginx/html/html8001;
    index index.html;
}

# 基于 IP
server{
    listen 80;
    server_name 112.74.164.244;
    root /usr/share/nginx/html/html8001;
    index index.html;
}

# 基于域名
server{
    listen 80;
    server_name nginx2.abc.com;
    location / {
        root /usr/share/nginx/html/html8001;
        index index.html index.htm;
    }
}
```
#### 5.3、反向代理
正向代理：把不让访问的服务器的网页请求，代理到一个可以访问该网站的代理服务器上来，一般叫做 proxy 服务器，再转发给客户    
反向代理：向外部客户端提供了一个统一的代理入口，客户端的请求都要先经过这个 proxy 服务器。使用反向代理客户端用户只能通过外来网来访问代理服务器，并且用户并不知道自己访问的真实服务器是那一台，可以很好的提供安全保护；主要用途是为多个服务器提供负债均衡、缓存等功能
```
# proxy_set_header :在将客户端请求发送给后端服务器之前，更改来自客户端的请求头信息
# proxy_connect_timeout:配置Nginx与后端代理服务器尝试建立连接的超时时间
# proxy_read_timeout : 配置Nginx向后端服务器组发出read请求后，等待相应的超时时间
# proxy_send_timeout：配置Nginx向后端服务器组发出write请求后，等待相应的超时时间
# proxy_redirect :用于修改后端服务器返回的响应头中的Location和Refresh
server{
    listen 80;
    server_name nginx2.abc.com;
    location / {
        proxy_pass http://abc.com;
    }
}
```
#### 5.4、适配 PC 或移动设备
```
location ~ ^/gzact/copyright/(.*)$ {  
    proxy_set_header Host   $host;
    proxy_set_header X-Real-IP      $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    if ($http_user_agent ~* "(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|mobile.+firefox|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows ce|xda|xiino") {
        set $mobile_rewrite "wap";
    }
    if ($mobile_rewrite != "wap") { 
        rewrite ^(/gzact/copyright)/(.*)  /gzact/copyright/pc/$2 break; 
        proxy_pass http://localhost:9091;
    }
    if ($mobile_rewrite = "wap") { 
        rewrite ^(/gzact/copyright)/(.*)  /gzact/copyright/wap/$2 break;
        proxy_pass http://localhost:9091;
    }
}
```
#### 5.5、Gzip 压缩配置
```
// gzip: 该指令用于开启或 关闭 gzip 模块
// gzip_buffers: 设置系统获取几个单位的缓存用于存储 gzip 的压缩结果数据流
// gzip_comp_level: gzip 压缩比，压缩级别是 1-9，1 的压缩级别最低，9 的压缩级别最高。压缩级别越高压缩率越大，压缩时间越长
// gzip_disable: 可以通过该指令对一些特定的 User-Agent 不使用压缩功能
// gzip_min_length: 设置允许压缩的页面最小字节数，页面字节数从相应消息头的 Content-length 中进行获取
// gzip_http_version：识别 HTTP 协议版本，其值可以是 1.1. 或 1.0.
// gzip_proxied: 用于设置启用或禁用从代理服务器上收到相应内容 gzip 压缩
// gzip_vary: 用于在响应消息头中添加 Vary：Accept-Encoding，使代理服务器根据请求头中的 Accept-Encoding 识别是否启用 gzip 压缩
// 配置成功后，HTTP 响应头 Content-Encoding 为 gzip 类型，或使用 gzip 在线检测网页查看。
http {
    .....
    gzip on;
    gzip_types text/plain application/javascript text/css;
    .....
}
```
#### 5.6、负载均衡
```
upstream webservers {
    server 172.18.144.23:4789 weight=10;
    server 172.18.144.23:5789 weight=10;
}
server {
    listen 8000;
    location / {
        proxy_pass http://myapp1;
    }
}
```
#### 5.7、获取真实 IP
```
server {
  listen 80;
  server_name bbs.expressjiaocheng.com;
  #charset koi8-r;
  #access_log /var/log/nginx/host.access.log main;
  location / {
    #设置主机头和客户端真实地址，以便服务器获取客户端真实 IP
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header X-Nginx-Proxy true;
    #禁用缓存
    proxy_buffering off;
    proxy_pass http://bakeaaa;
  }
}
```
#### 5.8、spa 刷新设置及二级目录
```
server {
  listen 8000;
  server_name localhost;
 
  location /dianping {
    root /usr/local/var/web;
    index index.html;
    try_files $uri /dianping/index.html;
  }

  location /mock {
    root /usr/local/var/web/dianping;
  }
}
```
#### 5.9、https 设置
相应 https 配置信息及文件需要去云服务器商去配置获取，具体信息见 [网络之 https 协议](/note/net/https/)
```
server {
  listen 80;
  server_name coupon.master-ss.cn;
  rewrite ^(.*) https://$host$1 permanent;
}

server {
  listen 443;
  server_name coupon.master-ss.cn;
  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/1_coupon.master-ss.cn_bundle.crt;
  ssl_certificate_key /etc/nginx/conf.d/ssl/2_coupon.master-ss.cn.key;
  ssl_session_timeout 5m;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;#按照这个套件配置
  ssl_prefer_server_ciphers on;
  
  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header X-Nginx-Proxy true;
   
    proxy_pass http://127.0.0.1:3001; 
    proxy_redirect off;
  }
}
```
#### 5.10、案例
```
upstream contentact_proxy {
  server 192.168.31.88:5697 max_fails=3 fail_timeout=5s;
}

location ~ ^/gzact/copyright/(.*)$ {  
  if ($http_user_agent ~* "(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|mobile.+firefox|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows ce|xda|xiino") {
    set $mobile_rewrite "wap";
  }
  if ($mobile_rewrite != "wap") { 
    rewrite ^(/gzact/copyright)/(.*)  /gzact/copyright/pc/$2 break; 
    proxy_pass http://contentact_proxy;
  }
  if ($mobile_rewrite = "wap") { 
    rewrite ^(/gzact/copyright)/(.*)  /gzact/copyright/wap/$2 break; 
    proxy_pass http://contentact_proxy;
  }
}

    
location ~ ^/gzact/(.*)\.(gif|jpg|jpeg|png|bmp|swf|ico)$ {
  access_log off;
  proxy_pass http://contentact_proxy;
}

location ~ ^/gzact/(.*)\.(js|css)$ {
  access_log off;
  proxy_pass http://contentact_proxy;
}
```