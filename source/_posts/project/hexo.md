---
title: hexo 之博客建站
categories:
- project 
---
主要总结分享下我用 [hexo](https://hexo.io/zh-cn/docs/) 建站的过程。
<!--more-->
### 一、基础
#### 1.1、概述
Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。
#### 1.2、建站
```
<!-- 安装全局依赖 -->
npm install -g hexo-cli

<!-- 初始化工程项目 -->
hexo init blog

<!-- 进入目录 -->
cd blog

<!-- 安装依赖 -->
npm i

<!-- 开启本地服务 -->
hexo s

<!-- 编译生成静态文件 -->
hexo g

<!-- 清除缓存 -->
hexo clean
```
### 二、进阶
#### 2.1、安装主题
默认主题是 landscape，这里选择 [indigo](https://github.com/yscoder/hexo-theme-indigo) 主题。
```
<!-- 安装主题 -->
git clone git@github.com:yscoder/hexo-theme-indigo.git themes/indigo

<!-- 安装主题依赖包 -->
npm install hexo-renderer-less --save
npm install hexo-generator-feed --save
npm install hexo-generator-json-content --save
npm install hexo-helper-qrcode --save

<!-- 开启分类页 -->
hexo new page categories

<!-- 修改 hexo/source/categories/index.md 数据 -->
layout: categories
comments: false
---
```
#### 2.2、配置主题
更详细配置信息参考[官方主题文档](https://github.com/yscoder/hexo-theme-indigo/wiki/%E9%85%8D%E7%BD%AE)
```
<!-- 编辑站点配置文件，/_config.yml，启用主题，且配置基础信息 -->
theme: indigo

<!-- 修改访问链接和目录格式 -->
url: http://blog.master-ss.cn
permalink: :title/

<!-- 配置主题 /themes/indigo/_config.yml -->
1、配置菜单
2、配置 favicon
3、配置头像
4、配置 email
5、替换打赏图片
6、配置 ‘ICP 备案号’
7、修改留言信息 postMessage
8、修改版权起始年份 since_year
9、启用 valine 评论插件
```
#### 2.3、部署
截止目前我们就可以在本地进行文档的书写了，但是想要 hexo d 就部署到 git 上还是不够，需要进行以下操作。
```
<!-- 编辑站点配置文件，/_config.yml，配置部署仓库、类型和分支 -->
deploy:
  type: git
  repo: https://github.com/zss007/blog.git
  branch: gh-pages

<!-- 安装部署依赖包 -->
npm install hexo-deployer-git –save
```
#### 2.4、注意事项
由于我们是使用 `git clone git@github.com:yscoder/hexo-theme-indigo.git themes/indigo` 方式进行主题安装，所以我们在提交代码到 git 时 `/themes/indigo` 文件夹中的所有文件均不会被提交。所以如果想刚才修改配置的主题信息同样提交到 git 上的话，可以 `cd /themes/indigo`，如果移除主题的仓库信息，即 `rm -rf .git`。