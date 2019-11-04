---
title: Linux 之文件及目录
categories:
- note 
---
这周把 Linux 基础总结了一下，以 Centos 为标准，先说文件和目录这块。
<!--more-->
### 一、基础知识
#### 1.1、shell 命令技巧
- tab 补全
  - 如果 1 次 tab 且文件或命令唯一存在，则补全代码
  - 如果 2 次 tab 则列出所有匹配到的文件或命令
- 上下键盘查看最近的历史命令

#### 1.2、目录结构介绍
<img src="/assets/note/linux/construct.png">
- root 目录：linxu 超级权限 root 的主目录
- home 目录：系统默认的用户主目录，如果添加用户是不指定用户的主目录，默认在`/home`
下创建与用户同名的文件夹
- bin 目录：存放系统所需要的重要命令，比如文件或目录操作的命令 ls、cp、mkdir 等，另外`/usr/bin`也放了一些系统命令。这些命令对应着文件都是可以执行的
- sbin 目录：存放只有 root 超级管理员才能执行的程序
- boot 目录：存放着 linux 启动时内核及引导系统程序所需要的核心文件，内核文件和 grub 系统引导管理器都位于此目录
- dev 目录：存放这 linux 系统下的设备文件，如光驱等

### 二、目录与路径
#### 2.1、目录相关操作
所有的目录下面都会存在两个目录，分别是“.”和“..”
```
.  代表此层目录
.. 代表上一层目录
-  代表前一个工作目录
~  代表“目前用户身份”所在的主文件夹
~account 代表 account 这个用户主文件夹(account 是个账号名称)
```
#### 2.2、处理目录
- cd: 切换目录

```
cd [相对路径或绝对路径]

<!--代表去到 vbird 这个用户的主文件夹，即 /home/vbird-->
cd ~vbird

<!--表示回到自己主文件夹，即 /root 目录-->
cd ~

<!--没有加上任何路径，也还是代表回到主文件夹-->
cd

<!--表示去到目前的上层目录，即 /root 的上层目录-->
cd ..

<!--表示回到刚刚的个目录-->
cd -

<!--绝对路径-->
cd /var/spool/mail

<!--相对路径-->
cd ../mqueue
```
- ls：查看文件与目录
当只执行 `ls` 时，默认显示的是只有非隐藏文件的文件名、以文件名进行排序及文件名代表的颜色显示

```
ls [-adl] 目录名称
a: 全部的档案，连同隐藏文件（开头为 . 的文件）一起列出来
d: 仅列出目录本身，而不是列出目录的文件数据
l: 长数据串，包含文件的属性与权限等数据
```
- pwd: 显示当前目录

```
pwd
```
- mkdir: 新建新目录

```
mkdir [-mp] 目录名称
<!---m: 配置文件目录权限，直接设定，不需要看默认权限 (umask)-->
<!---p: 帮你直接将所需要的目录(包括上层目录)递归建立-->

<!--建立名为 test 新目录-->
mkdir test

<!--加了 -p 的选项，可自行帮你建立多层目录-->
mkdir -p test1/test2/test3/test4

<!--建立权限为 rwx--x--x 的目录-->
mkdir -m 711 test2
```
- rmdir: 删除空目录，被删除的目录里面必定不能存在其他的目录或文件

```
rmdir [-p] 目录名称
<!---p: 连同上层“空的”目录也一起删除-->
```
#### 2.3、关于执行文件路径的变量：$PATH
```
<!--查询环境变量-->
echo $PATH
/root/.nvm/versions/node/v8.11.2/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```
### 三、文件与目录管理
- touch：创建新文件

```
touch 文件

<!--新建一个空的文件-->
touch testtouch
```
- cp：复制文件或目录
在默认条件中，cp 的源文件与目标文件的权限是不同的，目标文件的所有者通常会是命令操作者本身

```
cp [-ir] 源文件(source) 目标文件(destination)
-i: 若目标文件已存在时，在覆盖时会先询问操作的进行
-r: 递归持续复制，用于目录的复制行为

<!--用 root 身份，将主文件下的 .bashrc 复制到 /tmp 下，并更名为 bashrc-->
cp ~/.bashrc /tmp/bashrc

<!--复制到当前目录，最后的 . 不要忘-->
cp /var/log/wtmp .

<!--可以将多个数据一次复制到同一个目录去，最后面一定是目录-->
cp ~/.bashrc ~/.bash_history /tmp
```
- rm：移除文件或目录

```
rm [-fr] 档案或目录
-f: 就是 force，忽略不存在的文件，不会出现警告;
-r: 递归删除，常用在目录的删除

<!--透过通配符*的帮忙，将开头为 bashrc 文件名通通删除:-->
rm -i bashrc*

<!--将 /tmp/etc/ 这个目录删除掉-->
rm -r /tmp/etc
```
- mv：移动文件与目录，或更名

```
mv [-f] source destination
-f: force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖

<!--将目录名称更名为 mvtest2-->
mv mvtest mvtest2

<!--如果有多个源文件或目录，则最后一定是目录，即将所有的数据移动到该目录-->
mv bashrc1 bashrc2 mvtest2
```
### 四、文件内容查阅
- cat: 由第一行开始显示文件内容

```
cat [-n]
-n: 打印出行号，连同空白行也会有行号

<!--查看 /etc/issue 这个文件的内容，加印行号-->
cat -n /etc/issue
<!--返回结果-->
1 CentOS release 5.3 (Final)
2 Kernel \r on an \m
3
```
- tac: 从最后一行开始显示

```
tac /etc/issue

<!--返回-->
Kernel \r on an \m
CentOS release 5.3 (Final)
```
- more: 一页一页地显示文件内容

```
more master-stdout.log

Space：向下翻页
Enter：向下滚动一行
/字符串：代表在显示的内容中，向下查询“字符串”这个关键词
:f: 立即显示出文件名以及目前显示的行数
q: 立即离开more，不再显示该文件内容
```
- less：与 more 类似，可以往前翻页

```
Space: 向下翻页
Enter：向下滚动一行
PageDown【fn+上方向】：向下翻页
PageUp【fn+下方向】：向上翻页
/字符串：向下查询“字符串”的功能
?字符串：向上查询“字符串”的功能
q：离开less这个程序
```
- head：取出前面几行

```
head [-n number] 文件
-n: 后面接数字，代表显示几行

<!--默认的情况显示前十行，若要显示前 20 行，如下所示-->
head -n 20 /etc/man.config

<!--列出前面的所有行数，但不包括后面100行-->
head -n -100 /etc/man.config
```
- tail：取出后面几行

```
tail [-n number] 档案
-n :后面接数字，代表显示几行
-f :表示持续侦测后面的文件，要等到按下ctrl+c才会结束

<!--默认显示最后十行，若要显示最后 20 行，就得要这样-->
tail -n 20 /etc/man.config

<!--不知道/etc/man.config 有几行，只想列出 100 行以后的数据-->
tail -n +100 /etc/man.config

<!--持续侦测/var/log/messages 的内容-->
tail -f /var/log/messages
```
- |：管道命令
数据经过几道手续之后才能得到我们所想要的格式，使用`|`界定符号。每个管道后面接的必定是“命令”，而且这个命令能接受 standard input，如 less, more, head, tail 等都是可以接受 standard input 的管道命令
```
<!--使用ls命令输出后的内容能被less读取，并利用less功能翻动相关信息，按字母 ‘q’ 退出-->
ls -al /etc | less
```

- grep 选取命令
```
grep [-inv] [--color=auto] '查找字符串' filename
-i 忽略大小写
-n 输出行号
-v 反向选择
--color=auto 找到关键词部分加上颜色显示

<!--将 last 当中有 root 的那一行读取出来-->
last | grep 'root'

<!--只有没有 root 就取出来-->
last | grep -v 'root'

<!--取出 /etc/man.config 内行含 MAHPATH 的那几行-->
grep --color=auto 'MAHPATH' /etc/man.config
```
### 五、文件的查询
- which：寻找“执行文件”

```
which [-a] command
-a :将所有由 PATH 目录中可以找到的命令均列出，而不只第一个被找到的命令名称

<!--搜寻 ifconfig 指令完整文件名-->
which ifconfig
```
- whereis: 寻找特定文件
Linux 会将系统内的所有文件都记录在一个数据库文件里面，当使用 whereis 或 locate 时，以数据库文件为准。有时还能找到已经被删除的文件，而且找不到最新的刚创建的文件

```
<!--找出 nginx 这个文件名-->
whereis nginx
nginx: /usr/sbin/nginx /usr/lib64/nginx /etc/nginx /usr/share/nginx /usr/share/man/man8/nginx.8.gz
```
- locate: 找出关键词文件
输入“文件的部分名称”就能够得到结果

```
locate keyword
```
- find：磁盘查找文件

```
find [PATH] [option] [action]

<!--搜寻文件名为 filename 的文件-->
find / -name passwd
```
### 六、文件系统简单操作
- df: 列出文件系统的整体磁盘使用量

```
df [-h] [目录或文件名]
-h: 以人们较易阅读的 GBytes, MBytes, KBytes 等格式自行显示

<!--在 df 后面加上目录或是档案时，df 会自动分析该目录的文件所在分区 ，并将该 分区的容量显示出来-->
df -h /etc
```
- du: 评估文件系统的磁盘使用量
这个命令其实会直接到文件系统内去查找所有的文件数据，会执行段时间

```
du [-skm] 文件或目录名称
-s：列出总量而已，而不列出每个各别的目录占用容量
-k：以 KB 列出容量
-m：以 MB 列出容量

<!--检查当前目录下所有文件或文件夹所占容量-->
du -sm .
712     ./coupon
3       ./logs
1       ./server.js
```
