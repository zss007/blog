---
title: Linux 之用户管理与文件权限
categories:
- note 
---
以 Centos 为标准，这章说用户管理与文件权限。
<!--more-->
### 一、用户管理
- useradd 添加用户

```
useradd lisi
```
- passwd 设置密码

```
<!--后面没有加账号就是改自己的密码-->
passwd lisi
```
- userdel 删除用户

```
<!---r:递归的删除目录下面文件以及子目录下文件-->
userdel -r lisi
```
- id 查看用户

```
id user
```
- groups 查看用户组

```
<!--有效与支持用户组的查看-->
groups
```
- groupadd 新增用户组

```
<!--新建一个用户组，名为 group1-->
groupadd group1
```
- groupdel 删除用户组

```
<!--删除 mygroup 组，不能删除初始用户组-->
groupdel mygroup
```
- gpasswd 用户组管理

```
<!--把用户 testuser 加入到 root 组，加入组后 testuser 获取到 user 组及 root 组所有权限-->
gpasswd -a testuser root

<!--移出组-->
gpasswd -d testuser root
```
### 二、用户身份切换
- su 需新切换的用户密码

```
<!--切换 root 身份-->
su -

<!--切换 dmtsai 身份-->
su -l dmtsai
```
- sudo 仅需要自己的密码

```
<!--以 sshd 身份在 /tmp 下新建 mysshd 文件，不设置默认为 root-->
sudo -u sshd touch /tmp/mysshd

<!--查看当前用户可执行的命令-->
sudo -l
```
- visudo 为用户添加 sudo 权限

```
<!--输入-->
visudo

<!--编辑-->
%zhangsan ALL=(root) /usr/sbin/useradd
%zhangsan ALL=(root) /usr/sbin/userdel

<!--用户账号  登陆者来源主机名=（可切换的身份） 可执行的命令-->
<!--在用户账号最左边加上 % 代表后面接的是“用户组”-->
%admin    ALL = (ALL) ALL
```
### 三、文件权限
#### 3.1、介绍
```
<!--显示文件与相关属性-->
ls -al

-rw-r--r--  1 root root   331 Oct 28  2018 server.js
```
第一个字符代表文件类型：
- `d`为目录
- `-`为文件
- `l`为连接文件

- 接下来的字符串 3 个为一组，均为`rwx`组合，`r`代表可读，`w`代表可写，`x`代表可执行
  - 第一组为‘文件所有者的权限’
  - 第二组为‘同用户组的权限’
  - 第三组为‘其他非本用户组的权限’
- 第二列表示有多少个文件名连接到此节点
- 第三列表示这个文件的‘所有者账号’
- 第四列表示这个文件的所属用户组
- 第五列为这个文件的容量大小，默认为B
- 第六列为这个文件的创建或最近修改时间
- 第七列为该文件名

#### 3.2、改变文件属性与权限
- chgrp 改变文件所属用户组

```
chgrp [-R] dirname/filename
-R：进行递归更改，即连同子目录下的所有文件、目录都更新成为这个用户组，常用于更新某一目录内所有的文件情况

如修改用户组为 users
chgrp users install.log
```
- chown 改变文件所有者

```
chown [-R] 账号名称 文件或目录
chown [-R] 账号名称:组名 文件或目录

将 install.log 的拥有者改为 bin 这个账号
chown bin install.log

将 install.log 的拥有者和群组改回为 root
chown root:root install.log
```
- chmod 改变文件的权限
文件权限的改变使用的是 chmod 这个指令，但是权限的设置方法有两种， 分别可以使用数字或者
符号来进行权限的变更

  - 数字类型改变档案权限
r：4
w：2
x：1

  - 符号类型改变档案权限
u：user
g：group
o：others
a：all
+：加入
-：除去
=：设置

```
chmod [-R] xyz 文件或目录
xyz: 就是刚刚提到的数字类型的权限属性，为 rwx 属性数值的相加

如：将 .bashrc 文件的权限修改为-rw-r--r--
chmod 644 .bashrc

如：将 .bashrc 文件的权限修改为-rwxr-xr-x
chmod u=rwx,go=rx .bashrc

如：不知道原先的文件属性，只想增加.bashrc 这个文件的每个人均可写入的权限
chmod a+w .bashrc
```

#### 3.3、目录与文件的权限意义
- 权限对文件的重要性
  - r：读取文件的实际内容，如读取文本文件的文字内容等
  - w：编辑、新增、修改文件内容（不含删除）
  - x：具备被系统执行的权限
- 权限对目录的重要性
  - r：读取目录结构列表
  - w：更新目录结构列表的权限，如：新建文件或目录、删除文件或目录（不论该文件的权限）、重命名文件或目录、转移目录内文件或目录位置
  - x：代表用户能否进入该目录成为工作目录，所谓工作目录就是你目前所在的目录

要开放目录给任何人浏览时，应该至少也要给予 r 及 x 的权限，但 w 权限不可随便给