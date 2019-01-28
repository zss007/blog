---
title: node 之 npm
categories:
- node
---
npm 是 nodeJs 的包管理器，使 JavaScript 开发人员可以轻松地共享和重用代码，并且可以轻松更新您共享的代码。NodeJs 自带 npm，所以下载安装 NodeJs 即可。
<!--more-->
### 一、安装 npm 包
安装依赖包是 NPM 最常见的用法，执行安装命令后，NPM 会在当前目录下创建 node_modules 目录，然后在node_modules 目录下创建包目录，接着将包解压到这个目录下。
安装好依赖包后，直接在代码中调用 require('xxx') 即可引入该包。require() 方法在做路径分析的时候会通过模块路径查找到 xxx 所在的位置。模块引入和包的安装这两个步骤是相辅相承的。
```
// 本地安装
npm install <package_name>
// 全局安装
npm install <package_name> -g
// 添加依赖性
npm install <package_name> --save
npm install <package_name> --save-dev
```
有两种方法来安装 npm 包：本地或全局。您可以根据要使用的软件包选择要使用的安装类型：本地安装即类似 Node.js 的 require，在自己项目中安装然后引入使用；全局安装会将 npm 包下载安装到 node 安装文件夹中，然后作为命令行工具来使用。
添加依赖性也有两种方法 *--save* 和 *--save-dev*，分别添加到 package.json 的 *dependencies* (发布之后还依赖的东西) 和 *devDependencies* (开发时候依赖的东西)，具体使用哪个，取决于如何使用这种依赖性。比如，你写 ES6 代码，如果你想编译成 ES5 发布那么 babel 就是 devDependencies。如果你用了 jQuery，由于发布之后还是依赖 jQuery，所以是 dependencies。
- 从本地安装。对于一些没有发布到 NPM 上的包，或是因为网络原因导致无法直接安装的包，可以通过将包下载到本地，然后以本地安装。本地安装只需为 NPM 指明 package.json 文件所在的位置即可：它可以是一个包含 package.json 的存档文件，也可以是一个 URL 地址，也可以是一个目录下有 package.json 文件的目录位置。具体参数如下：

```
npm install <tarball file>
npm install <tarball url>
npm install <folder>
```
- 从非官方源安装。如果不能通过官方源安装，可以通过镜像源安装。在执行命令时，添加`--registry=http://registry.url`即可，示例如下：

```
npm install underscore --registry=http://registry.url
```
如果使用过程中几乎都采用镜像源安装，可以执行以下命令指定默认源：
```
npm config set registry http://registry.url 
```
### 二、使用 package.json
管理 npm 包的最好方法是创建一个 package.json 文件，可以用作项目所依赖的包的文档、指定项目可以使用的包的版本及重现项目构建。
````
// 启动命令行问卷，在启动命令的目录中创建一个 package.json
npm init
// 填写默认值
npm init --yes
````
CommonJS 为 package.json 文件定义了如下一些必需的字段：
- name。包名。规范定义它需要由小写的字母和数字组成，可以包含 .、_ 和 -，但不允许出现空格。包名必须是唯一的，以免对外公布时产生重名冲突的误解。除此之外，NPM 还建议不要在包名中附带上 node 或 js 来重复标识它是 JavaScript 或 Node 模块。
- description。包简介。
- version。版本号。一个语义化的版本号，通常为 major.minor.revision 格式。该版本号十分重要，常常用于一些版本控制的场合。
- keywords。关键词数组，NPM 中主要用来做分类搜索。一个好的关键词数组有利于用户快速找到你编写的包。
- maintainers。包维护者列表。每个维护者由 name、email 和 web 这 3 个属性组成。NPM 通过该属性进行权限认证。示例如下：
```
"maintainers": [
    {
        "name": "Jackson Tian",
        "email": "shyvo1987@gmail.com",
        "web": "http://html5ify.com"
    }
]
```
- contributors。贡献者列表。在开源社区中，为开源项目提供代码是经常出现的事情，如果名字能出现在知名项目的 contributors 列表中，是一件比较有荣誉感的事。列表中的第一个贡献应当是包的作者本人。它的格式与维护者列表相同。
- bugs。一个可以反馈 bug 的网页地址或邮件地址。
- licenses。当前包所使用的许可证列表，表示这个包可以在哪些许可证下使用。它的格式如下：
```
"licenses": [
    {
        "type": "GPLv2",
        "url": "http://www.example.com/licenses/gpl.html",
    }
]
```
- repositories。托管源代码的位置列表，表明可以通过哪些方式和地址访问包的源代码。
- dependencies。使用当前包所需要依赖的包列表。这个属性十分重要，NPM 会通过这个属性帮助自动加载依赖的包。
- homepage。当前包的网站地址。
- os。操作系统支持列表。这些操作系统的取值包括 aix、freebsd、linux、macos、solaris、vxworks、windows。如果设置了列表为空，则不对操作系统做任何假设。
- cpu。CPU 架构的支持列表，有效的架构名称有 arm、mips、ppc、sparc、x86和x86_64。同 os 一样，如果列表为空，则不对 CPU 架构做任何假设。
- engine。支持的 JavaScript 引擎列表，有效的引擎取值包括 ejs、flusspferd、gpsee、jsc、spidermonkey、narwhal、node 和 v8。
- builtin。标志当前包是否是内建在底层系统的标准组件。
- directories。包目录说明。
- implements。实现规范的列表。标志当前包实现了 CommonJS 的哪些规范。
- scripts。脚本说明对象。它主要被包管理器用来安装、编译、测试和卸载包。示例如下：
```
"scripts": { 
    "install": "install.js",
    "uninstall": "uninstall.js",
    "build": "build.js",
    "doc": "make-doc.js",
    "test": "test.js"
}
```
在包描述文件的规范中，NPM实际需要的字段主要有 name、version、description、keywords、repositories、author、bin、main、scripts、engines、dependencies、devDependencies。
与包规范的区别在于多了 author、bin、main 和 devDependencies 这 4 个字段，下面补充说明一下。
- author。包作者。
- bin。一些包作者希望包可以作为命令行工具使用。配置好 bin 字段后，通过npm install package_name -g 命令可以将脚本添加到执行路径中，之后可以在命令行中直接执行。前面的 node-gyp 即是这样安装的。通过 -g 命令安装的模块包称为全局模式。
- main。模块引入方法 require() 在引入包时，会优先检查这个字段，并将其作为包中其余模块的入口。如果不存在这个字段，require() 方法会查找包目录下的 index.js、index.node、index.json 文件作为默认入口。
- devDependencies。一些模块只在开发时需要依赖。配置这个属性，可以提示包的后续开发者安装依赖包。

### 三、常用命令
````
// 6.5.0 查看版本号
npm -v
// 更新 npm 包
npm install npm -g
// 安装 / 更新包
npm install XXX
// 更新所有本地包(如果是更新全局包，必须使用 -g)
npm update
// 卸载本地包(如果包在 package.json 中依赖性，必须使用 --save 或者 --save-dev 卸载，如安装的时候，全局包同理添加 -g)
npm uninstall/rm <package>
// 查看全局安装过的包
npm list -g --depth 0
// 创建 npm 用户
npm adduser
// 登录，在客户端上存储凭据
npm login
// 确保凭据存储在客户端上
npm config ls
// 发布包
npm publish
// 更新 npm 包版本号
npm version x.x.x
// 查看当前安装包是否已经过时了(wanted: 满足 package.json 条件的最新版本号，如果不满足则显示当前安装的版本号，latest: latest tag 所在版本号)
npm outdated
// 执行后可以使用 npm i project-name@tag 来安装这个版本，这等价于 npm i project@x.x.x
// 如果之前版本存在这个 tag，那么之前版本的 tag 会被删除，当前执行版本添加 tag
npm dist-tag add <package>@<version> tag
// 切记不能删除 latest，否则 npm 仓库会报错，可以使用 add 命令更新版本号
npm dist-tag rm <pkg> <tag>
// 列举出所有的 tag
npm dist-tag ls [<pkg>]
// 分析出当前路径下能够通过模块路径找到的所有包，并生成依赖树
npm ls
````