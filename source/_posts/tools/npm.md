---
title: tools 之 npm
categories:
- tools
---
npm 是 nodeJs 的包管理器，使 JavaScript 开发人员可以轻松地共享和重用代码，并且可以轻松更新您共享的代码。NodeJs 自带 npm，所以下载安装 NodeJs 即可。那为什么前端也需要知道 npm 呢，nodeJs 不是后台开发吗？其实前端工具 bower、gulp 等等前端工具都依赖于 npm，所以在这里介绍下 npm。
<!--more-->
### 一、安装 npm 包
```
//本地安装
npm install <package_name>
//全局安装
npm install <package_name> -g
//添加依赖性
npm install <package_name> --save
npm install <package_name> --save-dev
```
有两种方法来安装 npm 包：本地或全局。您可以根据要使用的软件包选择要使用的安装类型：本地安装即类似 Node.js 的 require，在自己项目中安装然后引入使用；全局安装会将 npm 包下载安装到 node 安装文件夹中，然后作为命令行工具来使用。
添加依赖性也有两种方法 *--save* 和 *--save-dev*，分别添加到 package.json 的 *dependencies* (发布之后还依赖的东西) 和 *devDependencies* (开发时候依赖的东西)，具体使用哪个，取决于如何使用这种依赖性。比如，你写 ES6 代码，如果你想编译成 ES5发布那么babel 就是 devDependencies。如果你用了 jQuery，由于发布之后还是依赖 jQuery，所以是 dependencies。
### 二、使用 package.json
管理 npm 包的最好方法是创建一个 package.json 文件，可以用作项目所依赖的包的文档、指定项目可以使用的包的版本及重现项目构建。
````
//启动命令行问卷，在启动命令的目录中创建一个 package.json
npm init
//填写默认值
npm init --yes
````
package.json内容介绍：
````
{
  "name"："my_package"， //包名
  "description"：""， //包的描述信息
  "version"："1.0.0"，//包的版本信息
  "main"："index.js"，//模块的入口文件
  "scripts"：{  //钩子，定义一些脚本操作，比如测试之类的(NodeJs)
    "test"："echo \"错误：未指定测试\"&&退出1"
  }，
  "keywords"：[]， //关键字集合
  "author"：""，   //包的作者
  "license"："ISC"，//证书
  "repository"：{  //存储地址信息
    "type"："git"，
    "url"："https://github.com/ashleygwilliams/my_package.git"
  }，
  "bugs"：{  //bug信息
    "url"："https://github.com/ashleygwilliams/my_package/issues"
  }，
  "homepage"："https://github.com/ashleygwilliams/my_package" //包的网站主页介绍
}}
````
### 三、常用命令
````
// 更新npm包
npm install npm -g
// 安装 / 更新包
npm install XXX
// 更新所有本地包(如果是更新全局包，必须使用-g)
npm update
// 卸载本地包(如果包在package.json中依赖性，必须使用--save或者--save-dev卸载，如安装的时候，全局包同理添加-g)
npm uninstall/rm <package>
// 查看全局安装过的包
npm list -g --depth 0
// npm包细节
repository / description / README.md
// 创建npm用户
npm adduser
// 登录，在客户端上存储凭据
npm login
// 确保凭据存储在客户端上
npm config ls
//发布包
npm publish
// 更新npm包版本号
npm version x.x.x
// 查看当前安装包是否已经过时了(wanted: 满足package.json条件的最新版本号，如果不满足则显示当前安装的版本号，latest: latest tag所在版本号)
npm outdated
// 执行后可以使用npm i project-name@tag来安装这个版本，这等价于npm i project@x.x.x
// 如果之前版本存在这个tag，那么之前版本的tag会被删除，当前执行版本添加tag
npm dist-tag add <package>@<version> tag
// 切记不能删除latest，否则npm仓库会报错，可以使用add命令更新版本号
npm dist-tag rm <pkg> <tag>
// 列举出所有的tag
npm dist-tag ls [<pkg>]
````