---
title: redis 简介
categories:
- note 
---
Redis 是使用 C 语言开发的一个高性能键值数据库。Redis 可以通过一些键值类型来存储数据，键值类型： String字符类型、Map散列类型、 List列表类型、Set集合类型、Sortedset有序集合类型
<!--more-->
### 一、应用场景
- 缓存，如数据查询、短连接、新闻内容、商品内容等等
- 分布式集群架构中的 session 分离
- 任务队列，如秒杀、抢购、12306等等
- 数据过期处理（可以精确到毫秒）

### 二、使用
#### 2.1、安装
- mac

```
<!-- 使用 homebrew 快捷安装 -->
brew install redis
brew services start/stop redis
```
- windows
windows 中下载安装就不介绍了，下载链接如下：

```
https://github.com/MicrosoftArchive/redis/releases
```
- centos

```
<!-- 查看软件包信息 -->
yum info redis

<!-- 安装redis安装包 -->
yum install redis

<!-- 启动redis -->
systemctl start redis

<!-- 设置redis开机启动 -->
systemctl enable redis
```
#### 2.2、进入命令行
```
<!-- 输入后回车即进入redis客户端 -->
redis-cli
```
### 三、类型
#### 3.1、字符串
```
<!-- 查看所有的 key -->
keys *

<!-- 普通设置 -->
set key value

<!-- 设置并加过期时间，表示 30 秒后过期 -->
set key value EX 30

<!-- 获取数据 -->
get key

<!-- 删除指定数据 -->
del key

<!-- 删除全部数据 -->
flushall

<!-- 查看类型 -->
type key

<!-- 设置过期时间，表示指定的 key 20 秒后过期 -->
expire key 20
```
#### 3.2、列表
Redis 列表是简单的字符串列表，按照插入顺序排序，你可以添加一个元素到列表的头部(左边)或者尾部(右边)
```
<!-- 列表右侧增加值 -->
rpush key value

<!-- 列表左侧增加值 -->
lpush key value

<!-- 右侧删除值 -->
rpop key

<!-- 左侧删除值 -->
lpop key

<!-- 获取数据，索引可以是负数，如：“-1”代表最后边的一个元素 -->
lrange key start stop

<!-- 删除指定数据 -->
del key

<!-- 删除全部数据 -->
flushall

<!-- 查看类型 -->
type key
```
#### 3.3、集合
Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据，它和列表的最主要区别就是没法增加重复值
```
<!-- 给集合增数据 -->
sadd key value

<!-- 删除集合中的一个值 -->
srem key value

<!-- 获取数据 -->
smembers key

<!-- 删除指定数据 -->
del key

<!-- 删除全部数据 -->
flushall
```
#### 3.4、哈希
hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象
```
<!-- 设置值 hmset -->
hmset zhangsan name "张三" age 20 sex "男"

<!-- 设置值 hset -->
hset zhangsan name "张三"

<!-- 获取数据 -->
hgetall key

<!-- 删除指定数据 -->
del key

<!-- 删除全部数据 -->
flushall
```
### 四、订阅发布
Redis 发布订阅（pub/sub）是一种消息通信模式：发送者（pub）发送消息，订阅者（sub）接收消息。 Redis 客户端可以订阅任意数量的频道。
下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：
<img src="/assets/note/redis/subscribe.png">
当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：
<img src="/assets/note/redis/publish.png">

代码示例如下：
```
<!-- 发布 -->
var redis = require('redis');
var client = redis.createClient(6379, 'localhost');
client.publish('testPublish', 'message from publish.js');

<!-- 订阅 -->
client.subscribe('testPublish');
client.on('message', function(channel, msg) {
  console.log('client.on message, channel:', channel, ' message:', msg);
});
```