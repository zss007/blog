---
title: mongodb 聚合查询核心
categories:
- database
---
mongodb aggregation 核心。
<!--more--> 

### 一、$group
通过指定的 _id 字段对输入文档进行分组（_id 是用来标识分组），并针对每个不同的分组输出文档。每个输出文档的 _id 字段均包含唯一的按值分组，输出文档还可以包含某些 accumulator 表达式值的计算字段，如：$sum、$avg、$min、$max	等。
```
<!-- 先分组统计，再根据 count 倒序排列 -->
db.movies.aggregate([
  {
    "$group": {
      "_id": "$year",
      "count": { "$sum": 1 }
    }
  },
  {
    "$sort": { "count": -1 }
  }
])

<!-- $group 的字段可以使用表达式 -->
db.movies.aggregate([
  {
    "$group": {
      "_id": {
        "numDirectors": {
          "$cond": [{ "$isArray": "$directors" }, { "$size": "$directors" }, 0]
        }
      },
      "numFilms": { "$sum": 1 },
      "averageMetacritic": { "$avg": "$metacritic" }
    }
  },
  {
    "$sort": { "_id.numDirectors": -1 }
  }
])

<!-- 先过滤，再对所有数据进行分组，计算平均值 -->
db.movies.aggregate([
  {
    "$match": { "metacritic": { "$gte": 0 } }
  },
  {
    "$group": {
      "_id": null,
      "averageMetacritic": { "$avg": "$metacritic" }
    }
  }
])
```

### 二、accumulator
使用 $sum、$avg、$min、$max	等，$map、$reduce 可用于更复杂的计算。
```
<!-- 使用 $reduce 计算 avg_high_tmp 最大值 -->
db.icecream_data.aggregate([
  {
    "$project": {
      "_id": 0,
      "max_high": {
        "$reduce": {
          "input": "$trends",
          "initialValue": -Infinity,
          "in": {
            "$cond": [
              { "$gt": ["$$this.avg_high_tmp", "$$value"] },
              "$$this.avg_high_tmp",
              "$$value"
            ]
          }
        }
      }
    }
  }
])

<!-- 可使用 $max 简化操作 -->
db.icecream_data.aggregate([{
  "$project": {
    "_id": 0,
    "max_high": {
      "$max": "$trends.avg_high_tmp"
    }
  }
}])

<!-- 分组计算总和 -->
db.icecream_data.aggregate([
  {
    "$project": {
      "_id": 0,
      "yearly_sales (millions)": { "$sum": "$trends.icecream_sales_in_millions" }
    }
  }
])

<!-- 应用实例 -->
对所有获得至少 1 项奥斯卡奖的影片，请计算 imdb.rating 的标准偏差，最高，最低和平均值。
提示-该系列中所有获得奥斯卡奖的电影都以类似于以下奖项之一的字符串开头：
比如：
Won 13 Oscars
Won 1 Oscar
数字精确到小数点后 4 位
db.movies.aggregate([
  {
    $match: {
      awards: /Won \d{1,2} Oscars?/
    }
  },
  {
    $group: {
      _id: null,
      highest_rating: { $max: "$imdb.rating" },
      lowest_rating: { $min: "$imdb.rating" },
      average_rating: { $avg: "$imdb.rating" },
      deviation: { $stdDevSamp: "$imdb.rating" }
    }
  }
])
```

### 二、$unwind
将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。只在数组中使用，在大集合中使用 $unwind 会有性能问题，可能导致内存超出，所以尽早使用 $match 过滤。
```
<!-- 找出 2010 到 2015 电影 imdb.rating 最高的 genres -->
db.movies.aggregate([
  {
    "$match": {
      "imdb.rating": { "$gt": 0 },
      "year": { "$gte": 2010, "$lte": 2015 },
    }
  },
  {
    "$unwind": "$genres"
  },
  {
    "$group": {
      "_id": {
        "year": "$year",
        "genre": "$genres"
      },
      "average_rating": { "$avg": "$imdb.rating" }
    }
  },
  {
    "$sort": { "_id.year": -1, "average_rating": -1 }
  },
  {
    "$group": {
      "_id": "$_id.year",
      "genre": { "$first": "$_id.genre" },
      "average_rating": { "$first": "$average_rating" }
    }
  },
  {
    "$sort": { "_id": -1 }
  }
])

<!-- 应用实例 -->
计算每个 cast 的参加电影数量，并获取每个 cast 的平均 imdb.rating。
出现在英语作为可用语言的电影次数最多的 cast，电影数量和平均 imdb.rating（精确到小数点后一位）是多少？
按照以下顺序和格式提供输入：
{ "_id": "First Last", "numFilms": 1, "average": 1.1 }
$trunc 将数字截断为整数或指定的小数位：{ $trunc : [ <number>, <place> ] }，place 默认为 0
$trunc: [1234.5678, 0] => 1234
$trunc: [1234.5678, -2] => 1200
$trunc: [1234.5678, 2] => 1234.56
<!-- 聚合查询如下： -->
db.movies.aggregate([
  {
    $match: {
      languages: "English"
    }
  },
  {
    $project: { _id: 0, cast: 1, "imdb.rating": 1 }
  },
  {
    $unwind: "$cast"
  },
  {
    $group: {
      _id: "$cast",
      numFilms: { $sum: 1 },
      average: { $avg: "$imdb.rating" }
    }
  },
  {
    $project: {
      numFilms: 1,
      average: {
        $divide: [{ $trunc: { $multiply: ["$average", 10] } }, 10]
      }
    }
  },
  {
    $sort: { numFilms: -1 }
  },
  {
    $limit: 1
  }
])
```

### 三、$lookup
对同一数据库中的未分片集合执行左外部联接。在每个输入文档中，$lookup 阶段都会添加一个新的数组字段，其元素是 "joined" 集合中的匹配文档，$lookup 阶段将这些经过重整的文档传递到下一个阶段。
```
db.air_alliances.aggregate([
  {
    "$lookup": {
      "from": "air_airlines",
      "localField": "airlines",
      "foreignField": "name",
      "as": "airlines"
    }
  }
]).pretty()

<!-- 应用实例 -->
在 air_alliances 中使用 Boeing 747 或 Airbus A380（在 air_routes 中分别为 747 和 380）最多的是哪个 alliance？
<!-- 聚合查询实例 -->
db.air_routes.aggregate([
  {
    $match: {
      airplane: /747|380/
    }
  },
  {
    $lookup: {
      from: "air_alliances",
      foreignField: "airlines",
      localField: "airline.name",
      as: "alliance"
    }
  },
  {
    $unwind: "$alliance"
  },
  {
    $group: {
      _id: "$alliance.name",
      count: { $sum: 1 }
    }
  },
  {
    $sort: { count: -1 }
  }
])
```
### 四、$graphLookup
对集合执行递归搜索，其中包含用于通过递归深度和查询限制搜索的选项，from 为递归匹配 connectFromField 和 connectToField 字段的集合。
注意事项：
可能返回过多数据，导致内存溢出，使用 $allowDiskUse 选项；
或者给 connectToField 添加索引；
不能在分片集合中使用；

```
<!-- 自递归查询出所有下属 -->
db.parent_reference.aggregate([{
  $match: {
    name: 'Eliot'
  }
}, {
  $graphLookup: {
    from: 'parent_reference',
    startWith: '$_id',
    connectFromField: '_id',
    connectToField: 'reports_to',
    as: 'all_reports'
  }
}]).pretty()

<!-- 自递归查询出所有下属，maxDepth 如果为 0 则是直接下属，没有递归查询；depthField  -->
db.child_reference.aggregate([{
  $match: {
    name: 'Dev'
  }
}, {
  $graphLookup: {
    from: 'child_reference',
    startWith: '$direct_reports',
    connectFromField: 'direct_reports',
    connectToField: 'name',
    as: 'till_2_level_reports',
    maxDepth: 1,
    depthField: 'level'
  }
}]).pretty()

<!-- 跨表递归查询，restrictSearchWithMatch 用于递归查询过滤 -->
db.air_airlines.aggregate([{
  $match: {
    name: 'TAP Portugal'
  }
}, {
  $graphLookup: {
    from: 'air_routes',
    startWith: '$base',
    connectFromField: 'dst_airport',
    connectToField: 'src_airport',
    as: 'chain',
    maxDepth: 1,
    depthField: 'level',
    restrictSearchWithMatch: {
      'airline.name': 'TAP Portugal'
    }
  }
}], {
  allowDiskUse: true
}).pretty()

<!-- 应用实例 -->
查找最多有一个中转的，属于 "OneWorld" 联盟的航空公司的 base 机场的所有可能目的地的列表。航空公司应是德国，西班牙或加拿大的国家航空公司。包括目的地以及该位置的航空公司。小提示，应该找到158个目的地。
db.air_alliances.aggregate([
  {
    $match: { name: "OneWorld" }
  },
  {
    $graphLookup: {
      startWith: "$airlines",
      from: "air_airlines",
      connectFromField: "name",
      connectToField: "name",
      as: "airlines",
      maxDepth: 0,
      restrictSearchWithMatch: {
        country: { $in: ["Germany", "Spain", "Canada"] }
      }
    }
  },
  {
    $graphLookup: {
      startWith: "$airlines.base",
      from: "air_routes",
      connectFromField: "dst_airport",
      connectToField: "src_airport",
      as: "connections",
      maxDepth: 1
    }
  },
  {
    $project: {
      validAirlines: "$airlines.name",
      "connections.dst_airport": 1,
      "connections.airline.name": 1
    }
  },
  { $unwind: "$connections" },
  {
    $project: {
      isValid: {
        $in: ["$connections.airline.name", "$validAirlines"]
      },
      "connections.dst_airport": 1
    }
  },
  { $match: { isValid: true } },
  {
    $group: {
      _id: "$connections.dst_airport"
    }
  }
])
```