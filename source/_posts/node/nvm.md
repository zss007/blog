---
title: nvm 简介
categories:
- node
---
前不久有个旧项目需要低版本的 node 才能跑起来，但是新项目又需要新版本，还好有个工具叫 nvm 能够切换不同版本的 node。
<!--more--> 
### 一、安装
#### 1.1、mac 或 linux
安装命令：
```
curl https://raw.githubusercontent.com/cnpm/nvm/master/install.sh | bash
```
安装完成后关闭终端，重新打开终端输入 nvm 验证一下是否安装成功，当出现 "Node Version Manager" 时，说明已安装成功。
#### 1.2、windows
nvm-windows 最新下载地址 `https://github.com/coreybutler/nvm-windows/releases`，这里有四个可下载的文件：
- nvm-noinstall.zip： 这个是绿色免安装版本，但是使用之前需要配置
- nvm-setup.zip：这是一个安装包，下载之后点击安装，无需配置就可以使用，方便
- Source code(zip)：zip 压缩的源码
- Sourc code(tar.gz)：tar.gz 的源码，一般用于 *nix 系统

### 二、基本命令
- nvm install <version>：可以是 node.js 版本或最新稳定版本 latest
- nvm list：列出已经安装的 node.js 版本
- nvm uninstall <version>：卸载指定版本的 nodejs
- nvm use [version]：切换到使用指定的 nodejs 版本
- nvm alias default v9.3.0：给 node 版本设置别名
- nvm --version：显示当前运行的 nvm 版本

### 三、不同版本的 Nodejs 共享全局的 npm
用 nvm 管理 node 版本，会碰到这样一个问题：对于各个版本的全局 npm 模块，是各自独立的。因此，当你在 6.16.0 下全局安装了某个模块，然后切换到 8.14.1 之后又得重新安装。所以，解法就是 npm prefix：
```
// 获取当前的 prefix
npm config get prefix   // ~/.nvm/versions/node/v8.14.1

// 将 prefix 设置到一个全局目录下，比如新建一个 /Users/songsong.zhang/npm-global, 这个文件不要放在需要 sudo 的文件夹下
npm config set prefix /Users/songsong.zhang/npm-global
```
设置之后，再用 npm 安装全局模块时就会放在 npm-global 下，注意 npm/cnpm 的 prefix 是各自独立的，因此每个都需要设置一下。
然后呢，全局模块的可执行文件也会放在 npm-global/bin 目录下，想要执行这些命令的话，还需要添加一条 PATH，打开 .bashrc，末尾添加一行：
```
export PATH=/Users/songsong.zhang/npm-global/bin:$PATH
```
如果是 windows 的话，则需要配置环境变量。
### 四、使用 .nvmrc 文件
如果你的 node 版本与项目所需的版本不同，则可在项目根目录或其任意父级目录中创建 .nvmrc 文件，在文件中指定使用的 node 版本号，例如：
```
cat .nvmrc
// 6.16.0

nvm use
// Found '/Users/songsong.zhang/study/es6test/.nvmrc' with version <6.16.0>
// Now using node v6.16.0 (npm v3.10.10)

node -v
// v6.16.0
```