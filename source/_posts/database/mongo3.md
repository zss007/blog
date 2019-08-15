---
title: mongodb 之查询操作符
categories:
- database
---
介绍 mongodb 查询时常用的操作符。
<!--more--> 
### 一、比较操作符
- $in
包含在数组中
db.movieDetails.find({rated: {$in: ["G", "PG", "PG-13"]}}, {_id: 0, title: 1, rated: 1}).pretty()

- $gte & $lt
$gte 大于等于（greater than or equal to）；$lt 小于（less than）
db.movieDetails.find({runtime: {$gte: 90, $lt: 120}}, {_id: 0, title: 1, runtime: 1})

- $ne
不等于，包括完全不包含这个字段的情况
db.movieDetails.find({rated: {$ne: "UNRATED"}}, {_id: 0, title: 1, rated: 1})

### 二、元素操作符
- $exists
查找某个字段是否存在
db.moviesDetails.find({mpaaRating: {$exists: true}})
null 会查找值为 null 或不存在该字段的文档
db.movieDetails.find({mpaaRating: null})

- $type
查看符合类型的文档
db.movies.find({viewerRating: {$type: "int"}}).pretty()

### 三、逻辑操作符
- $or
查看满足单个条件
```
db.movieDetails.find({$or: [{"tomato.meter": {$gt: 95}},                               
                            {"metacritic": {$gt: 88}}]},
                     {_id: 0, title: 1, "tomato.meter": 1, "metacritic": 1})
```

- $and
查看同时满足多个条件，一般情况不需要，跟默认查询情况效果相同。但是可应用于多个条件限制在同一个字段上时，默认查询情况不能用重复的 key
```
db.movieDetails.find({$and: [{"metacritic": {$ne: null}},
                             {"metacritic": {$exists: true}}]},
                          {_id: 0, title: 1, "metacritic": 1})

// 能很好的查询值为 null 的情况，因为查询 null 包括值为 null 和键值对不存在的情况
db.movieDetails.find({$and: [{"metacritic": null},
                             {"metacritic": {$exists: true}}]},
                     {_id: 0, title: 1, "metacritic": 1})
```

### 四、数组操作符
- $all
数组中的所有元素均出现文档值中，被包含关系
```
db.movieDetails.find({genres: {$all: ["Comedy", "Crime", "Drama"]}}, 
                     {_id: 0, title: 1, genres: 1}).pretty()
```

- $size
查询符合指定长度数组的文档
db.movieDetails.find({countries: {$size: 1}}).pretty()

- $elemMatch
遍历对象数组或二维数组中至少有一个对象或数组符合的文档；如果不使用 $elemMatch，则完全遍历，不考虑对象或数组为整体
```
<!-- 二维数组 -->
{ _id: 1, results: [ 82, 85, 88 ] }
{ _id: 2, results: [ 75, 88, 89 ] }
<!-- 数据查询 -->
db.scores.find(
   { results: { $elemMatch: { $gte: 80, $lt: 85 } } }
)
<!-- 返回结果：因为 82 既大于等于 80 又小于 85 -->
{ "_id" : 1, "results" : [ 82, 85, 88 ] }

<!-- 对象数组 -->
boxOffice: [ { "country": "USA", "revenue": 228.4 },
             { "country": "Australia", "revenue": 19.6 },
             { "country": "UK", "revenue": 33.9 },
             { "country": "Germany", "revenue": 16.2 },
             { "country": "France", "revenue": 19.8 } ]
db.movieDetails.find({boxOffice: {$elemMatch: {"country": "Germany", "revenue": {$gt: 16}}}})
```

### 五、评估操作符
- $regex
正则表达式匹配文档
db.movieDetails.find({"awards.text": {$regex: /^Won.* /}}, {_id: 0, title: 1, "awards.text": 1}).pretty()

- $text
文本搜索匹配文档
```
{
  $text:
    {
      $search: <string>,
      $language: <string>,
      $caseSensitive: <boolean>,
      $diacriticSensitive: <boolean>
    }
}

db.articles.find( { $text: { $search: "coffee" } } )
<!-- 查询包含 bake、coffee、cake 的语句，甚至这些词的动词格式 -->
db.articles.find( { $text: { $search: "bake coffee cake" } } )
<!-- 查询确切 coffee shop 字符串 -->
db.articles.find( { $text: { $search: "\"coffee shop\"" } } )
<!-- 查询包含 bake，不包含 shop 的语句，甚至这些词的动词格式 -->
db.articles.find( { $text: { $search: "coffee -shop" } } )
<!-- $language 用来表达特定语言下的动词格式，标识为 none 为不识别动词 -->
<!-- $caseSensitive 是否大小写敏感 -->
db.articles.find( { $text: { $search: "Coffee", $caseSensitive: true } } )
```