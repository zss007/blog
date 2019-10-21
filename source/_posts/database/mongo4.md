---
title: mongodb 聚合查询基础
categories:
- database
---
mongodb aggregation 基础。
<!--more--> 
### 一、基础知识
- 远程连接
mongo "mongodb://cluster0-shard-00-00-jxeqq.mongodb.net:27017,cluster0-shard-00-01-jxeqq.mongodb.net:27017,cluster0-shard-00-02-jxeqq.mongodb.net:27017/test?replicaSet=Cluster0-shard-0" -authenticationDatabase admin -ssl -username m001-student -password m001-mongodb-basics

- 导出
mongodump -u m121 -p aggregations -h "Cluster0-shard-0/cluster0-shard-00-00-jxeqq.mongodb.net:27017,cluster0-shard-00-01-jxeqq.mongodb.net:27017,cluster0-shard-00-02-jxeqq.mongodb.net:27017" --db aggregations --authenticationDatabase admin --ssl -o /Users/songsong.zhang/Desktop/mongo

- 导入
mongorestore --db aggregations /Users/songsong.zhang/Desktop/mongo/aggregations

- 管道通信
$match > $project > $group
$match: 用来过滤文档查询
$project: 用来格式化输出
$group: 用来数组分组

### 二、基础命令
- $match
$match 使用 Mongodb 查询语句，应尽可能在前面被调用
```
db.solarSystem.aggregate([{
  "$match": { "type": { "$ne": "Star"} }
}, {
  "$count": "planets"
}]);

应用实例：
在下一个电影之夜帮助 MongoDB 挑选电影，可能的电影必须满足以下条件：
imdb.rating 至少为 7；
genres 不包含 "Crime" 或 "Horror"；
rated 为 "PG" 或 "G"；
languages 包含 "English" 和 "Japanese"；
具体管道通信代码如下：
var pipeline = [
  {
    $match: {
      "imdb.rating": { $gte: 7 },
      genres: { $nin: [ "Crime", "Horror" ] } ,
      rated: { $in: ["PG", "G" ] },
      languages: { $all: [ "English", "Japanese" ] }
    }
  }
]
```

- $project
一旦表明某个字段显式返回，则所有返回的字段均需要显式表明，_id 除外；
不仅仅可以移除和返回字段，而且可以添加新字段；
在聚合管道中可多次使用；
可使用已存在字段重新分配新值或创建新字段；
```
db.solarSystem.aggregate([{
	"$project": {
		"_id": 0,
		"name": 1,
		"myWeight": {
			"$multiply": [{
				"$divide": ["$gravity.value", 9.8]
			}, 86]
		}
	}
}]);

应用实例：
我们的电影数据集包含许多不同的文档，其中一些具有比其他更复杂的标题。如果我们想分析集合以查找仅由一个单词组成的电影标题，则可以获取数据集中的所有电影并在客户端应用程序中进行一些处理，但是 aggregation 允许我们在数据库中处理！
查找标题由一个单词组成的电影数量，比如，"Cinderella" 和 "3-25" 应该算在内，而 "Cast Away" 则不算。
db.movies.aggregate([
  {
    $match: {
      title: {
        $type: "string"
      }
    }
  },
  {
    $project: {
      title: { $split: ["$title", " "] },
      _id: 0
    }
  },
  {
    $match: {
      title: { $size: 1 }
    }
  }
]).itcount()
```

- $addFields
为查询结果添加字段，如果已存在则覆盖原字段。$project 有同样的效果，但是通常用于显示或移除字段。
```
db.solarSystem.aggregate([
{"$project": {
    "_id": 0,
    "name": 1,
    "gravity": 1,
    "mass": 1,
    "radius": 1,
    "sma": 1}
},
{"$addFields": {
    "gravity": "$gravity.value",
    "mass": "$mass.value",
    "radius": "$radius.value",
    "sma": "$sma.value"
}}]);
```

- 指针类
$limit、$skip、$count、$sort
```
db.solarSystem.aggregate([{
  "$match": {
    "type": "Terrestrial planet"
  }
}, {
  "$count": "terrestrial planets"
}]).pretty();

使用实例：
MongoDB安排了另一个电影之夜。这次，我们对员工进行了投票，以找出他们最喜欢的女演员或演员，并获得了这些结果：
favorites = [
  "Sandra Bullock",
  "Tom Hanks",
  "Julia Roberts",
  "Kevin Spacey",
  "George Clooney"
]
对于在 USA 发行的 Tomatos.viewer.rating 大于或等于 3 的电影，请计算一个名为 num_favs 的新字段，该字段表示在该电影的 cast 字段中出现多少个 favorites。对 num_favs，tomatos.viewer.rating 和 title 进行降序排序，结果中第 25 部电影的标题是什么？
聚合查询如下：
db.movies.aggregate([{
  "$match": {
    "tomatoes.viewer.rating": { "$gte": 3 },
    "countries": "USA",
    "cast": {
      "$in": favorites
    }
  }
}, {
  "$project": {
    "_id": -1,
    "title": 1,
    "tomatoes.viewer.rating": 1,
    "num_favs": {
      "$size": {
        "$setIntersection": [
          "$cast",
          favorites
        ]
      }
    }
  }
}, {
  "$sort": {
    "num_favs": -1,
    "tomatoes.viewer.rating": -1,
    "title": -1
  }
}, {
  "$skip": 24
}, {
  "$limit": 1
}])
```

- 综合实例
计算每部可选语言是英语的电影的平均评分，而且 imdb.rating 至少为 1，imdb.votes 至少为 1，并且该电影于 1990 年或之后发行。需要重新规范化 imdb.votes，公式如下：
scaled_votes = 1 + 9 * ((x - x_min) / (x_max - x_min))
x_max = 1521105
x_min = 5
x = imdb.votes
normalized_rating = average(scaled_votes, imdb.rating)
哪部电影的 normalized_rating 最低？
```
聚合查询如下：
db.movies.aggregate([
  {
    $match: {
      year: { $gte: 1990 },
      languages: { $in: ["English"] },
      "imdb.votes": { $gte: 1 },
      "imdb.rating": { $gte: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      title: 1,
      "imdb.rating": 1,
      "imdb.votes": 1,
      normalized_rating: {
        $avg: [
          "$imdb.rating",
          {
            $add: [
              1,
              {
                $multiply: [
                  9,
                  {
                    $divide: [
                      { $subtract: ["$imdb.votes", 5] },
                      { $subtract: [1521105, 5] }
                    ]
                  }
                ]
              }
            ]
          }
        ]
      }
    }
  },
  { $sort: { normalized_rating: 1 } },
  { $limit: 1 }
])
```