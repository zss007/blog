---
title: Linux 之文件压缩、编辑与别名管理
categories:
- note 
---
以 Centos 为标准，这章说文件压缩、编辑与别名管理。
<!--more-->
### 一、文件系统的压缩打包
- zip & unzip

```
<!--安装 zip 减压软件-->
yum install -y unzip zip

<!---r：递归，表示将指定的目录下的所有子目录以及文件一起处理-->
zip -r public.zip(压缩包名称) ./public(压缩目录路径)

unzip ./public.zip(压缩包路径) [-d dir]
```
- tar

```
<!--制作 gz 包-->
tar czvf public.tar.gz ./public

<!--解压 gz 包-->
tar xzvf public.tar.gz

<!--查看 gz 包-->
tar tf public.tar.gz

<!--制作 tar 包，仅打包，不压缩！-->
tar cvf wwwroot.tar wwwroot

<!--减压 tar 包-->
tar xvf wwwroot.tar

特别注意，在参数的下达中，c/x/t 仅能存在一个！不可同时存在！因为不可能同时压缩与解压缩。
-c：建立一个压缩档案的参数指令(create 的意思)
-x：解开一个压缩档案的参数指令
-t：查看 tarfile 里面的文件

-z：是否同时具有 gzip 的属性？亦即是否需要用 gzip 压缩？
-j：是否同时具有 bzip2 的属性？亦即是否需要用 bzip2 压缩？
-v：压缩的过程中显示档案！这个常用，但不建议用在背景执行过程！
-f：使用文件名，请留意，在 f 之后要立即接文件名喔！不要再加参数！
```
### 二、vi/vim 快速入门
vi 是 Unix 和类 Unix 环境下的可用于创建文件的屏幕编辑器。vi 有两种工作模式：命令模式和文本输入模式。vim 是 vi 的升级版本，它不仅兼容 vi 的所有指令，而且还有一些新的特性在里面。

vi 命令模式：
默认编辑一个文件的时候第一次进入的就是命令模式，vi 从命令模式切换到文本输入模式可以在键盘上面按(i 或者 I 或者 a 或者 A 或者 O 或者 o)，按[ESC]键使 vi 从文本输入模式 回到命令模式。
<img src="/assets/note/linux/vi1.png">

退出 vi 命令模式：
<img src="/assets/note/linux/vi2.png">

命令模式下面文本修改键：
<img src="/assets/note/linux/vi3.png">
<img src="/assets/note/linux/vi4.png">

### 三、命名别名设置
- alias（仅在当前终端有效）

```
<!-- alias 后面加 {"别名"='命令参数...'}，如： -->
alias ll='ls -l'

<!--查看目前所有的命名别名-->
alias
```
- unalias（仅在当前终端有效）

```
<!--取消别名-->
unalias ll
```
### 四、环境变量设置别名
- `/etc/profile` 和 `/etc/bashrc` 是 系统整体的设置，最好不要修改
- `~/.bash_history` 记录之前输入命令
- `~/.bash_logout` 退出时执行的命令
- `~/.bash_profile` 登入shell时执行
- `~/.bashrc` 非登入shell，一般被显式 `~/.bash_profile` 调用
- source 读入环境配置文件，使其生效，而不必重启

```
<!--source 配置文件名-->
source ~/.bashrc

<!-- 比如可在 `~/.bash_profile` 文件编辑添加配置文件使别名永久生效 -->
# User specific aliases and functions
alias ll='ls -l'
```