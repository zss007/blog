---
title: tools 之 bower
categories:
- tools
---
Bower 是 twitter 推出的包管理工具，支持包的下载、更新、卸载等操作。
<!--more-->
### 一、为什么使用 Bower   
以往，我们要使用 jQuery、Bootstrap 等模块，会到网上进行搜索，找到下载地址，然后通过浏览器或其它工具下载下来使用。Bower 提供了更简单的获取模块方式。只要一条命令，需要的资源就下载下来了。
Bower 是命令行工具，通过 npm 安装。
```
npm install -g bower
```
这行命令是 Bower 的全局安装，-g 操作表示全局。Bower 依赖 node, npm 和 git。
### 二、开始使用
#### 包的安装
通过命令 *bower install* 安装软件包，默认会安装到 *bower_components/目录*.
```
bower install <package>
```
想要下载的包可以是 GitHub 上的短链接（如 jquery/jquery）、 .git 、 一个 URL 或者其它. 
```
# 通过 bower.json 文件安装
bower install
# 通过在 github 上注册的包名安装
bower install jquery
# GitHub 短链接（github 别名自动解析 git 库）
bower install desandro/masonry
# Github上的 .git
bower install git://github.com/user/package.git
# URL
bower install http://example.com/script.js
# 下载本地库
bower install ./repos/jquery
```
### 三、新建 bower.json
可以通过 bower init 命令新建一个 bower.json 文件。会提示你输入一些基本信息，根据提示按回车或者空格即可，然后会生成一个 bower.json 文件，用来保存该项目的配置，如下：
```
{
  "name": "blue-leaf",  //包名，必选。
  "version": "1.0.0",  //有意义的版本号。
  "description": "Physics-like animations for pretty particles", //描述
  "main": [  //字符串或者数组，指定主要会用到包里面哪些文件(不会直接用到)
    "js/motion.js",
    "sass/motion.scss"
  ],
  "dependencies": {  //哈希结构，依赖的包，可以指定版本号
    "get-size": "~1.2.2",
    "eventEmitter": "~4.2.11"
  },
  "devDependencies": { //哈希结构，生产环境下依赖的包
    "qunit": "~1.16.0"
  },
  "moduleType": [ //模块类型
    "amd",
    "globals",
    "node"
  ],
  "keywords": [ //关键字(便于bower搜索查找)
    "motion",
    "physics",
    "particles"
  ],
  "authors": [ //作者
    "Betty Beta <bbeta@example.com>"
  ],
  "license": "MIT",  //证书
  "ignore": [ //忽略包的文件夹
    "**/.*",
    "node_modules",
    "bower_components",
    "test",
    "tests"
  ],
  "private": true //布尔值，设置为true代表你想保持私有，并且将来不会发布它
}
```
如果想保存依赖信息(dependencies)到你的 bower.json 文件，可以使用：bower install PACKAGE --save,或者使用 bower install <package> --save-dev 保存依赖信息(devDependencies).
### 四、使用下载的包
对于已经下载下来的包，默认在当前目录的 bower_components 文件夹。你可以直接在项目里引用。例如：
```
<script src="bower_components/jquery/dist/jquery.min.js"></script>
```
推荐将 Bower 与其它工具一起使用。
### 五、常用命令
```
bower help //命令查看帮助
bower install jquery --save //添加依赖并更新 bower.json 文件
bower cache clean //安装失败清除缓存
bower install storejs //安装 storejs
bower uninstall storejs //卸载 storejs
bower info jquery //包的信息
bower update //包的更新(根据 bower.json 文件中的版本号更新)
bower search bootstrap //包的查找
bower uninstall jquery //包的卸载
```
### 六、注册
#### 添加配置文件(即新建 bower.json)
#### 注册自己的包
可以注册自己的包，这样其他人也可以使用了，这个操作只是在服务器上保存了一个映射，服务器本身不托管代码。
```
bower register storejs git://github.com/jaywcjlove/store.js.git
```
### 七、配置
```
{
  "cwd": "~/my-project",  //当前工作目录，运行bower的目录
  "directory": "bower_components",  //自定义bower下载的代码包的目录
  "registry": "https://bower.herokuapp.com", //注册表配置,默认为bower注册表URL
  "shorthand-resolver": "git://github.com/{{owner}}/{{package}}.git",  //为速记包名称定义自定义模板,定义Bower在为给定速记构造URL时使用的自定义模板的支持
  "proxy": "http://proxy.local",  //设置http代理
  "https-proxy": "https://proxy.local", //设置https代理
  "ca": "/var/certificate.pem", //要使用的CA证书，默认为null
  "color": true, //启用或禁用在CLI输出中使用颜色
  "timeout": 60000, //发出请求时使用的超时值
  "strict-ssl": true, //是否在通过https发出请求时执行SSL密钥验证
  "storage": { //在哪里存储持久数据，如缓存，bower需要
    "packages" : "~/.bower/packages",
    "registry" : "~/.bower/registry",
    "links" : "~/.bower/links"
  },
  "tmp": "〜/.bower/tmp" //在哪里存储临时文件和文件夹。默认为系统临时目录后缀/bower
  "interactive": true, //使bower交互，在必要时提示
  "resolvers": [  //用于定位和提取包的Pluggable Resolver列表
    "mercurial-bower-resolver"
  ],
  "shallowCloneHosts": [ //允许将已知支持浅克隆的主机列入白名单
    "myGitHost.example.com"
  ],
  "scripts": { //脚本,可用于在Bower使用期间触发其他自动化工具,这些钩子旨在允许外部工具帮助将新安装的组件连接到父项目和其他类似的任务中
    "preinstall": "",
    "postinstall": "",
    "preuninstall": ""
  },
  "ignoredDependencies": [ //当解析包时，Bower将忽略这些依赖关系
    "jquery"
  ]
}
```
Bower 的配置文件是 .bowerrc，使用 JSON 格式进行描述，如上。