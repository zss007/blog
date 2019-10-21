---
title: mongodb 之 Compass
categories:
- database
---
对 mongodb 做一个系统性的研究和总结，先从可视化工具 MongoDB Compass 开始。
<!--more--> 
### 一、下载
#### 1.1、简介
MongoDB Compass 是官方数据库可视化工具，下载地址是 https://www.mongodb.com/download-center/compass 。
#### 1.2、连接
输入数据库和用户信息，连接数据库。
<img src="/assets/database/mongo/compass-login.png">

连接成功后即可显示数据库信息。
<img src="/assets/database/mongo/database.png">
### 二、分析
#### 2.1、查看集合
选择查看的集合，点击即可查看
<img src="/assets/database/mongo/collection.png">

#### 2.2、分析结构
可点击切换到 Schema，查看分析其结构，或者查看索引等。
<img src="/assets/database/mongo/schema.png">

#### 2.3 条件查询
用户可自主输入搜索条件或者在 Schema 中点击，即可获取搜索条件。
<img src="/assets/database/mongo/search.png">

用户甚至可以使用 shift + 拖拽 的方式进行地理位置区域查询。
<img src="/assets/database/mongo/search-geo.png">