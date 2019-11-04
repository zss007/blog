---
title: Linux 之软件安装
categories:
- note 
---
以 Centos 为标准，这章说软件安装。
<!--more-->
### 一、软件管理器
- rpm
全名“RedHat Package Manager”，简称 RPM，以数据库记录的方式将所需要的软件安装到 Linux 系统。最大特点是将要安装的软件先编译过，并且打包成 RPM 机制的安装包，通过软件里头默认的数据库记录要安装的时候必须具备的依赖属性软件。

```
rpm -ivh package_name
-i：install 的意思
-v：查看更详细的安装信息画面
-h：以安装信息栏显示安装进度

<!--安装 rp-pppoe-3.5-32.1.i386.rpm-->
rpm -ivh rp-pppoe-3.5-32.1.i386.rpm

<!--直接由网络上面的某个档案安装，以网址来安装-->
rpm -ivh http://website.name/path/pkgname.rpm

<!--列出所有的，已经安装在本机 Linux 系统上面的所有软件名称-->
rpm -qa

<!--仅查询，后面接的软件名称是否有安装-->
rpm -q 软件名称

<!--httpd 表示要卸载的软件包-->
rpm -e httpd
```
- yum：rpm 属性依赖解决方式
依赖属性的软件列表，在有要安装软件需求的时候先到列表去找，同时与系统内已安装的软件相比较，没安装到的依赖软件就一口气同时安装起来，这就是 YUM 机制的由来。
想要使用 yum 功能，必须先找到适合的 yum server 才行，而每个 yum server 可能都会提供不同的软件功能

```
<!--yum 搜索 rpm 包-->
yum search 名称

<!--yum 显示 rpm 包信息-->
yum info package1

<!--yum 查看 rpm 包-->
yum list
yum list | grep httpd
<!--列出所有可更新的软件包-->
yum list updates
<!--列出所有已安装的软件包-->
yum list installed

<!--yum 卸载 rpm 包-->
yum [-y] remove wget
-y：当 yum 要等用户输入时，自动提供 yes 的响应

<!--yum 安装 rpm 包-->
yum [-y] install unzip zip

<!--后面接要升级的软件，若要整个系统都升级，直接 update 即可-->
yum [-y] update [软件名]
```
- yum 的设置文件

`ll /etc/yum.repos.d/`返回相关文件信息，然后动态设置部分软件的 yum 的镜像站点
```
total 44
-rw-r--r--. 1 root root  614 Oct 25  2018 CentOS-Base.repo
-rw-r--r--  1 root root 1309 Oct  8  2018 CentOS-CR.repo
-rw-r--r--  1 root root  649 Oct  8  2018 CentOS-Debuginfo.repo
-rw-r--r--  1 root root  230 Oct 25  2018 CentOS-Epel.repo
-rw-r--r--  1 root root  314 Oct  8  2018 CentOS-fasttrack.repo
-rw-r--r--  1 root root  630 Oct  8  2018 CentOS-Media.repo
-rw-r--r--  1 root root 1331 Oct  8  2018 CentOS-Sources.repo
-rw-r--r--  1 root root 4768 Oct  8  2018 CentOS-Vault.repo
-rw-r--r--  1 root root  196 Oct 31  2018 mongodb.repo
-rw-r--r--  1 root root   99 Jan  4  2019 nginx.repo
```
### 二、源码与 Tarball
- make
执行时在当前目录下搜索 Makefile 文本文件，这个文件记录了源码如何编译，make 自动判别源码是否经过变动而自动更新执行文件

- 源代码（C语言）如何编译安装

```
<!--先安装源代码编译的软件 gcc，make，openssl-->
yum install -y gcc make gcc-c++ openssl-devel

<!--检查系统中是否已经安装 gcc-->
rpm -qa | grep gcc / rpm -ql gcc

<!--1.生成编译配置文件(Makefile)-->
<!--2.开始编译(make)-->
<!--3.开始安装(make install)-->
安装 httpd-2.2.9.tar.gz 源代码:
1) 减压并 cd 到对应目录
2) ./configure --prefix=/usr/local/apache（安装路径设置为/usr/local/apache）
3) make / make -j4
4) make install
```
- 卸载源代码安装的软件

```
<!-- 1、结束进程，pkill 与 killall 差不多 -->
pstree | grep httpd
pkill httpd
<!-- 2、删除源代码 -->
cd /usr/local/
直接删除源代码
rm -rf apache/
```
- 源代码安装 nodejs

```
1、 下载 nodejs 源码包
2、 减压到 usr/local/nodejs 目录
3、 ./configure
4、 make / make -j4
5、 make install
```
- 源代码安装 Apache

```
1.减压 httpd-2.2.9.tar.gz 到对应目录
2、 ./configure 编译
./configure --prefix=/usr/local/apache2/ --sysconfdir=/usr/local/apache2/etc/ --with-included-apr --enable-dav --enable-so --enable-deflate=shared --enable-expires=shared
--enable-rewrite=shared
3、make
4、make install
<!--启动 Apache 测试:-->
/usr/local/apache2/bin/apachectl restart
<!--查看进程:-->
ps -le | grep httpd
```
- 二进制安装配置 nodejs
二进制包里面包括了已经经过编译，可以马上运行的程序，所以二进制包的安装只需要丢到一个目录里面

```
wget https://nodejs.org/dist/v8.9.3/node-v8.9.3-linux-x64.tar.xz

xz -d node-v8.9.3-linux-x64.tar.xz
tar -xvf node-v8.9.3-linux-x64.tar

mv node-v8.9.3-linux-x64 /usr/local/nodejs

<!--配置环境变量-->
vi /etc/profile

<!--最后面添加:-->
export NODE_HOME=/usr/local/nodejs/bin
export PATH=$NODE_HOME:$PATH

<!--:wq 保存，然后运行-->
source /etc/profile

<!--用 node -v 和 npm -v 来检查下:-->
node -v

<!--查看环境变量是否生效-->
echo $PATH
```
