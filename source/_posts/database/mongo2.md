---
title: mongodb 基础
categories:
- database
---
mongodb shell 安装和基础操作。
<!--more--> 
### 一、安装
- 安装 homebrew
`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

- 安装 mongodb
brew install mongodb

- 开机自启动
brew services start mongodb

### 二、更新
- 更新 homebrew
brew update

- 更新 mongodb
brew upgrade mongodb

- 查询 mongodb 相关软件
brew search mongodb

- 查询 mongodb 相关信息
brew info mongodb

- 卸载 mongodb
brew uninstall mongodb

### 三、基础命令
- 进入命令行
mongo

- 连接远程数据库
mongo "mongodb://cluster0-shard-00-00-jxeqq.mongodb.net:27017,cluster0-shard-00-01-jxeqq.mongodb.net:27017,cluster0-shard-00-02-jxeqq.mongodb.net:27017/test?replicaSet=Cluster0-shard-0" --authenticationDatabase admin --ssl --username m001-student --password m001-mongodb-basics

- 退出命令行
quit()

- 查看数据库列表
show databases

- 使用数据库、创建数据库
use databases

- 查看当前数据库
db

- 删除当前数据库
db.dropDatabase()

- 不切换当前数据库访问其他的数据库
db.getSiblingDB('name')

- 删除集合
db.collection.drop()

- 查看集合数据数量
db.collection.count()

- 插入单条数据
db.collection.insertOne(doc)

- 批量插入
db.collection.insertMany([doc...], { "ordered": false })
ordered 为 false 即遇到出错继续执行插入操作，默认为 true，遇错停止

- 数据查询
db.collection.find({a: b, "c.d": e, "f.0": g, h: [i, j]}).pretty()
查询同时满足多个条件的数据；
pretty() 格式化输出；
如果 "a" 对应的值是数组类型，则查找文档中 "a" 数组含有 b 元素；
"c.d" 是嵌套文档；
查找文档中 "f" 数组第一个是 g 元素；
查找文档中 "h" 数组含 [i, j] 且数组元素顺序固定；

- 查询下一页
it
命令行查找默认显示20条数据，如果查看更多输入 it（iterate）遍历下个20条数据

- 分页查询
sort: 1 升序；-1 降序
skip: 查询 5 条之后的数据
limit: 总计返回 10 条件数据
db.userInfo.find().sort({age: -1}).skip(5).limit(10)

- 格式化输出字段
db.collection.find({a: b}, {c: 1})
查找默认显示全部字段，如果 project 特别指定返回某些字段，如 c: 1，则显示 c 和 _id，其他字段隐藏，_id 需明确指定（_id : 0）不返回才不会显示；如果设定一些字段为 0，则隐藏这些字段，其他字段显示

- 查找集合中某个字段所有不同的值
```
<!-- 数据对象 -->
{ "_id": 1, "dept": "A", "item": { "sku": "111", "color": "red" }, "sizes": [ "S", "M" ] }
{ "_id": 2, "dept": "A", "item": { "sku": "111", "color": "blue" }, "sizes": [ "M", "L" ] }
{ "_id": 3, "dept": "B", "item": { "sku": "222", "color": "blue" }, "sizes": "S" }
{ "_id": 4, "dept": "A", "item": { "sku": "333", "color": "black" }, "sizes": [ "S" ] }
<!-- 字段返回 -->
db.inventory.distinct( "dept" )
[ "A", "B" ]
<!-- 嵌套字段返回 -->
db.inventory.distinct( "item.sku" )
[ "111", "222", "333" ]
<!-- 数组返回 -->
db.inventory.distinct( "sizes" )
[ "M", "S", "L" ]
<!-- 加查询条件 -->
db.inventory.distinct( "item.sku", { dept: "A" } )
[ "111", "333" ]
```

- 替换某条数据
updateOne 是更新文档中某个字段，replaceOne 则是替换整个文档
```
let filter = {title: "House, M.D., Season Four: New Beginnings"}
let doc = db.movieDetails.findOne(filter);
doc.poster;
doc.poster = "https://www.imdb.com/title/tt1329164/mediaviewer/rm2619416576";
doc.genres;
doc.genres.push("TV Series");
db.movieDetails.replaceOne(filter, doc);
```

- 删除某条数据
db.orders.deleteOne( { "_id" : ObjectId("563237a41a4d68582c2509da") } )
deleteOne 删除单文档，deleteMany 删除多文档

- 执行 js 文件
load('filePath.js')

### 四、更新操作符
- $set
db.collection.updateOne({a: b}, {$set: {c: d}})
更新单条数据，c 的值将被更新为 d，updateMany 更新所有匹配到的数据

- $push & $each
$push 将元素添加到数组中；$each 遍历数组，添加到 reviews 数组中；upsert 选项为若没有查询到匹配的文档，则插入该文档
```
db.movieDetails.updateOne({
  title: "The Martian"
}, {
  $push: {
    reviews: {
      $each: [{
         rating: 0.5,
         reviewer: "Yabo A."
      }, {
         rating: 5,
         reviewer: "Kristina Z."
      }]
    }
  }
}, {
  upsert: true
})
```

- $unset
删除相应字段
```
db.movieDetails.updateMany({
  rated: null
}, {
  $unset: {
    rated: ""
  }
})
```

- $rename
db.students.updateMany( {}, { $rename: { "nmae": "name" } } )
更新字段名

- $inc
$inc 字段增加值，接受正负数，如果字段不存在则为设置值，如果值为 null 则报错
```
db.products.update(
   { sku: "abc123" },
   { $inc: { quantity: -2, "metrics.orders": 1 } }
)
```

- $addToSet
添加值到数组中，含去重，即相同元素不添加
```
db.inventory.update(
   { _id: 2 },
   { $addToSet: { tags: { $each: [ "camera", "electronics", "accessories" ] } } }
 )
```

- $pop
从数组中移除第一个或最后一个元素，其中 -1 为第一个，1 为最后一个
db.students.update( { _id: 1 }, { $pop: { scores: -1 } } )

- $pull
从数组中移除满足条件的元素
```
db.stores.update(
    { },
    { $pull: { fruits: { $in: [ "apples", "oranges" ] }, vegetables: "carrots" } },
    { multi: true }
)
```

- $pullAll
从数组中移除匹配到的值
db.survey.update( { _id: 1 }, { $pullAll: { scores: [ 0, 5 ] } } )

